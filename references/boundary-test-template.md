# 边界测试模板

阶段 3 的参考文件。提供 KNOWN_VIOLATIONS 格式、棘轮逻辑和测试骨架。

## KNOWN_VIOLATIONS 格式

文件：`tests/architecture/known-violations.json`

```json
[
  {
    "file": "src/components/UserCard.tsx",
    "line": 5,
    "imports": "src/services/userService",
    "from_layer": "components",
    "to_layer": "services",
    "reason": "历史遗留耦合 —— 计划在 Q2 移除"
  }
]
```

**规则：**
- 每条违规一个条目。每条条目由 `file` + `imports` 唯一标识。
- 基线建立后，条目只能**删除**（修复），不能**新增**。
- 如果出现不在列表中的新违规，棘轮测试失败。
- 违规修复后，删除对应条目。数量只能缩小。

## 层级定义

在测试文件或配置文件中定义每层允许的 import：

```json
{
  "types": [],
  "lib": ["types"],
  "services": ["lib", "types"],
  "components": ["lib", "types"],
  "pages": ["components", "services", "lib", "types"]
}
```

每个键是一个层级，其值列出它可以 import 的层级。任何超出此范围的 import 都是违规。

---

## 测试骨架：TypeScript（Jest/Vitest）

> **测试名称提示：** 以下测试描述使用中文，对大多数现代 CI 系统（GitHub Actions、GitLab CI）无影响。如果你的 CI 系统（如老版 Jenkins）对非 ASCII 字符有解析问题，将 `describe`/`test` 中的中文改为英文即可。

```typescript
import { readdirSync, readFileSync } from 'fs';
import { join, relative } from 'path';
import knownViolations from './known-violations.json';

const LAYER_RULES: Record<string, string[]> = {
  types: [],
  lib: ['types'],
  services: ['lib', 'types'],
  components: ['lib', 'types'],
  pages: ['components', 'services', 'lib', 'types'],
};

// 匹配任意行中的 `from '...'` —— 处理单行和多行 import，
// 以及重导出（`export { x } from '...'`）。
// 注意：这是简化版扫描器。如需完整精度（动态 import、require()），
// 请使用 ts-morph 或 @babel/parser 进行 AST 解析（参见 references/stack-routing.md）。
const FROM_RE = /\bfrom\s+['"]([^'"]+)['"]/;

function getLayer(filePath: string): string | null {
  // 将 'src' 适配为项目实际的源码根目录
  const match = filePath.match(/^src\/([^/]+)\//);
  if (match && match[1] in LAYER_RULES) return match[1];
  return null;
}

function resolveTargetLayer(importPath: string): string | null {
  // 去除常见路径别名（@/、~/、#/）
  const normalized = importPath.replace(/^[@~#]\//, '');
  const segments = normalized.split('/');
  for (const layer of Object.keys(LAYER_RULES)) {
    if (segments.includes(layer)) return layer;
  }
  return null;
}

function scanFile(filePath: string): Array<{ file: string; line: number; imports: string; from_layer: string; to_layer: string }> {
  const violations: Array<{ file: string; line: number; imports: string; from_layer: string; to_layer: string }> = [];
  const content = readFileSync(filePath, 'utf-8');
  const lines = content.split('\n');
  const fromLayer = getLayer(relative(process.cwd(), filePath));
  if (!fromLayer) return violations;

  let inTypeImport = false;
  for (let i = 0; i < lines.length; i++) {
    const line = lines[i];

    // 追踪类型专用 import（编译时擦除，不是运行时依赖）
    if (/^\s*import\s+type\s/.test(line)) inTypeImport = true;

    const match = line.match(FROM_RE);
    if (match && !inTypeImport) {
      const targetLayer = resolveTargetLayer(match[1]);
      if (targetLayer && !LAYER_RULES[fromLayer].includes(targetLayer) && targetLayer !== fromLayer) {
        violations.push({
          file: relative(process.cwd(), filePath),
          line: i + 1,
          imports: match[1],
          from_layer: fromLayer,
          to_layer: targetLayer,
        });
      }
    }

    // 当 from 子句结束语句时，重置类型 import 追踪
    if (inTypeImport && FROM_RE.test(line)) inTypeImport = false;
  }
  return violations;
}

// 递归收集所有源文件
function collectFiles(dir: string, ext: string[]): string[] {
  const results: string[] = [];
  for (const entry of readdirSync(dir, { withFileTypes: true })) {
    const fullPath = join(dir, entry.name);
    if (entry.isDirectory()) {
      results.push(...collectFiles(fullPath, ext));
    } else if (ext.some(e => entry.name.endsWith(e))) {
      results.push(fullPath);
    }
  }
  return results;
}

describe('Architecture Boundary', () => {
  // 将 'src' 适配为项目实际的源码根目录
  const files = collectFiles('src', ['.ts', '.tsx']);
  const allViolations = files.flatMap(scanFile);

  test('no new violations', () => {
    const knownSet = new Set(knownViolations.map(v => `${v.file}:${v.imports}`));
    const newViolations = allViolations.filter(v => !knownSet.has(`${v.file}:${v.imports}`));

    if (newViolations.length > 0) {
      const msg = newViolations
        .map(v => `VIOLATION: ${v.file}:${v.line} imports ${v.imports} — ${v.from_layer} 不能导入 ${v.to_layer}。参见 docs/architecture/LAYERS.md`)
        .join('\n');
      throw new Error(`发现新的架构违规：\n${msg}`);
    }
  });

  test('violation count only shrinks', () => {
    expect(allViolations.length).toBeLessThanOrEqual(knownViolations.length);
  });
});
```

---

## 测试骨架：Python（pytest）

```python
import ast
import json
from pathlib import Path

LAYER_RULES = {
    "models": [],
    "config": ["models"],
    "db": ["config", "models"],
    "services": ["db", "config", "models"],
    "middleware": ["services", "config", "models"],
    "routes": ["services", "middleware", "models"],
}

KNOWN_VIOLATIONS_PATH = Path("tests/architecture/known-violations.json")


def get_layer(file_path: Path) -> str | None:
    # 将 'src' 适配为项目实际的源码根目录（例如包名、'app/'）
    parts = file_path.parts
    if len(parts) >= 2 and parts[0] == "src" and parts[1] in LAYER_RULES:
        return parts[1]
    return None


def scan_imports(file_path: Path) -> list[dict]:
    violations = []
    source = file_path.read_text()
    try:
        tree = ast.parse(source)
    except SyntaxError:
        return violations  # 跳过无法解析的文件
    from_layer = get_layer(file_path)
    if not from_layer:
        return violations

    def _check_target(target: str, node: ast.AST) -> None:
        for layer in LAYER_RULES:
            if layer in target.split("."):
                if layer != from_layer and layer not in LAYER_RULES[from_layer]:
                    violations.append({
                        "file": str(file_path),
                        "line": node.lineno,
                        "imports": target,
                        "from_layer": from_layer,
                        "to_layer": layer,
                    })
                break  # 只匹配第一个（最浅的）层级

    for node in ast.walk(tree):
        if isinstance(node, ast.Import):
            for alias in node.names:
                _check_target(alias.name, node)
        elif isinstance(node, ast.ImportFrom) and node.module:
            _check_target(node.module, node)

    return violations


def test_no_new_violations():
    known = json.loads(KNOWN_VIOLATIONS_PATH.read_text()) if KNOWN_VIOLATIONS_PATH.exists() else []
    known_set = {(v["file"], v["imports"]) for v in known}

    # 将 'src' 适配为项目实际的源码根目录（例如包名、'app/'）
    all_violations = []
    for py_file in Path("src").rglob("*.py"):
        all_violations.extend(scan_imports(py_file))

    new_violations = [v for v in all_violations if (v["file"], v["imports"]) not in known_set]
    assert not new_violations, "\n".join(
        f"VIOLATION: {v['file']}:{v['line']} imports {v['imports']} — "
        f"{v['from_layer']} 不能导入 {v['to_layer']}。参见 docs/architecture/LAYERS.md"
        for v in new_violations
    )


def test_ratchet_only_shrinks():
    known = json.loads(KNOWN_VIOLATIONS_PATH.read_text()) if KNOWN_VIOLATIONS_PATH.exists() else []

    all_violations = []
    for py_file in Path("src").rglob("*.py"):
        all_violations.extend(scan_imports(py_file))

    assert len(all_violations) <= len(known), (
        f"违规数量增加：{len(all_violations)} > 基线 {len(known)}。"
        "请修复违规以减少数量 —— 永远不要新增违规。"
    )
```

---

## 测试骨架：Go

```go
package architecture_test

import (
    "encoding/json"
    "go/parser"
    "go/token"
    "os"
    "path/filepath"
    "strings"
    "testing"
)

// 层级规则：key 层可以 import value 中的层，import 其他层均为违规
var layerRules = map[string][]string{
    "models":     {},
    "config":     {"models"},
    "db":         {"config", "models"},
    "services":   {"db", "config", "models"},
    "middleware": {"services", "config", "models"},
    "routes":     {"services", "middleware", "models"},
}

type Violation struct {
    File      string `json:"file"`
    Line      int    `json:"line"`
    Imports   string `json:"imports"`
    FromLayer string `json:"from_layer"`
    ToLayer   string `json:"to_layer"`
    Reason    string `json:"reason,omitempty"`
}

func getLayer(filePath string) string {
    // 将 "src" 适配为项目实际的源码根目录（Go module 包路径的第一级）
    parts := strings.Split(filepath.ToSlash(filePath), "/")
    for i, p := range parts {
        if p == "src" && i+1 < len(parts) {
            candidate := parts[i+1]
            if _, ok := layerRules[candidate]; ok {
                return candidate
            }
        }
    }
    return ""
}

func scanFile(filePath string) []Violation {
    var violations []Violation
    fset := token.NewFileSet()
    f, err := parser.ParseFile(fset, filePath, nil, parser.ImportsOnly)
    if err != nil {
        return violations // 跳过无法解析的文件
    }
    fromLayer := getLayer(filePath)
    if fromLayer == "" {
        return violations
    }
    allowed := layerRules[fromLayer]
    for _, imp := range f.Imports {
        path := strings.Trim(imp.Path.Value, `"`)
        for layer := range layerRules {
            if strings.Contains(path, "/"+layer+"/") || strings.HasSuffix(path, "/"+layer) {
                if layer == fromLayer {
                    break
                }
                isAllowed := false
                for _, a := range allowed {
                    if a == layer {
                        isAllowed = true
                        break
                    }
                }
                if !isAllowed {
                    pos := fset.Position(imp.Path.Pos())
                    violations = append(violations, Violation{
                        File:      filePath,
                        Line:      pos.Line,
                        Imports:   path,
                        FromLayer: fromLayer,
                        ToLayer:   layer,
                    })
                }
                break
            }
        }
    }
    return violations
}

func loadKnownViolations(t *testing.T) []Violation {
    t.Helper()
    data, err := os.ReadFile("known-violations.json")
    if os.IsNotExist(err) {
        return nil
    }
    if err != nil {
        t.Fatalf("读取 known-violations.json 失败: %v", err)
    }
    var known []Violation
    if err := json.Unmarshal(data, &known); err != nil {
        t.Fatalf("解析 known-violations.json 失败: %v", err)
    }
    return known
}

func TestArchitectureBoundary(t *testing.T) {
    var allViolations []Violation
    // 将 "src" 适配为项目实际的源码根目录
    _ = filepath.Walk("../../src", func(path string, info os.FileInfo, err error) error {
        if err != nil || info.IsDir() || !strings.HasSuffix(path, ".go") {
            return nil
        }
        allViolations = append(allViolations, scanFile(path)...)
        return nil
    })

    known := loadKnownViolations(t)
    knownSet := make(map[string]bool)
    for _, v := range known {
        knownSet[v.File+":"+v.Imports] = true
    }

    var newViolations []Violation
    for _, v := range allViolations {
        if !knownSet[v.File+":"+v.Imports] {
            newViolations = append(newViolations, v)
        }
    }

    if len(newViolations) > 0 {
        for _, v := range newViolations {
            t.Errorf("VIOLATION: %s:%d imports %s — %s 不能导入 %s。参见 docs/architecture/LAYERS.md",
                v.File, v.Line, v.Imports, v.FromLayer, v.ToLayer)
        }
    }

    if len(allViolations) > len(known) {
        t.Errorf("违规数量增加：%d > 基线 %d。棘轮只能缩小，请修复违规。", len(allViolations), len(known))
    }
}
```

---

## 为现有仓库建立基线

**步骤：**

1. 在没有 `known-violations.json` 的情况下运行边界测试，它会报告所有当前违规
2. 将输出整理为 JSON 格式保存为基线文件
3. 提交基线 —— 这是棘轮的起点
4. 从此开始，数量只能减少

**各技术栈建立基线的参考命令：**

```bash
# TypeScript（Vitest）
npx vitest run tests/architecture/boundary.test.ts 2>&1 \
  | grep "VIOLATION:" \
  | node scripts/violations-to-json.js \
  > tests/architecture/known-violations.json

# TypeScript（Jest）
npx jest tests/architecture/boundary.test.ts 2>&1 \
  | grep "VIOLATION:" \
  | node scripts/violations-to-json.js \
  > tests/architecture/known-violations.json

# Python
pytest tests/architecture/test_boundary.py -v 2>&1 \
  | grep "VIOLATION:" \
  | python scripts/violations_to_json.py \
  > tests/architecture/known-violations.json

# Go
go test ./tests/architecture/... -v 2>&1 \
  | grep "VIOLATION:" \
  | go run scripts/gc/violations_to_json.go \
  > tests/architecture/known-violations.json
```

> **最简单的做法：** 直接让测试在首次运行时自动生成 `known-violations.json`（当文件不存在时），而不是手动捕获输出。上方 Python 和 Go 骨架已内置此逻辑（`if KNOWN_VIOLATIONS_PATH.exists()`）。
