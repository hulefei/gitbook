# Lua 回调参数存储结构设计

在 Lua 中，当 `parameters` 的结构因不同回调类型动态变化时，可以通过以下方法获取其内部变量名和值：

---

## 方法 1：动态遍历键值对（通用方案）

直接遍历 `parameters` 表的所有键，动态获取所有变量名：

```lua
function printParameters(result)
    print("Event Type:", result.event_type)
    print("Parameters:")
    for key, value in pairs(result.parameters) do
        print("  " .. key .. " = " .. tostring(value))
    end
end

-- 示例输出：
-- Event Type: Download
-- Parameters:
--   task_id = 1001
--   file_id = abc
--   success = false
--   error_code = 404
```

---

## 方法 2：元数据映射（结构化方案）

为不同事件类型预定义参数结构，实现动态校验和文档化：

### 步骤 1：定义元数据映射

```lua
local PARAM_SCHEMA = {
    Init = { "success", "error_code" },          -- Init 回调的预期参数
    Download = { "task_id", "file_id", "success", "error_code" }  -- Download 回调的预期参数
}
```

### 步骤 2：动态获取参数并校验

```lua
function analyzeParameters(result)
    local event_type = result.event_type
    local params = result.parameters

    -- 获取实际存在的参数键
    local actual_keys = {}
    for k in pairs(params) do
        table.insert(actual_keys, k)
    end

    -- 对比预期和实际参数
    local expected_keys = PARAM_SCHEMA[event_type] or {}
    print("Expected Parameters:", table.concat(expected_keys, ", "))
    print("Actual Parameters:", table.concat(actual_keys, ", "))

    -- 检查缺失参数
    for _, key in ipairs(expected_keys) do
        if params[key] == nil then
            print("[WARNING] Missing parameter:", key)
        end
    end
end

-- 示例输出：
-- Expected Parameters: task_id, file_id, success, error_code
-- Actual Parameters: task_id, file_id, success, error_code
```

---

## 方法 3：统一接口封装（面向对象方案）

通过封装工具函数统一管理参数访问：

```lua
local ResultManager = {}

function ResultManager:new(event_type, parameters)
    local obj = {
        event_type = event_type,
        parameters = parameters
    }
    setmetatable(obj, self)
    self.__index = self
    return obj
end

-- 获取参数键列表
function ResultManager:getParamKeys()
    local keys = {}
    for k in pairs(self.parameters) do
        table.insert(keys, k)
    end
    return keys
end

-- 使用示例
local downloadResult = ResultManager:new("Download", {
    task_id = 1001,
    file_id = "abc",
    success = true,
    error_code = 0
})

print("Parameters:", table.concat(downloadResult:getParamKeys(), ", "))
-- 输出: Parameters: task_id, file_id, success, error_code
```

---

## 最佳实践建议

1. **混合使用动态和静态检查**

   - 用 `pairs` 遍历动态获取实际参数
   - 用预定义的元数据映射 (`PARAM_SCHEMA`) 验证完整性

2. **统一命名规范**  
   例如将 `bSuccess` 和 `bIsSuccess` 统一为 `success`，避免歧义

3. **添加调试工具函数**  
   类似 `printParameters` 的函数可快速诊断参数问题

4. **错误处理增强**

   ```lua
   function safeGetParam(result, key)
       local value = result.parameters[key]
       if value == nil then
           error(string.format("Invalid parameter key '%s' for event type '%s'", key, result.event_type))
       end
       return value
   end

   -- 安全访问示例
   local task_id = safeGetParam(downloadResult, "task_id")
   ```

---

通过结合动态遍历和静态元数据映射，既能灵活处理不同回调的参数差异，又能通过结构化检查确保数据完整性。
