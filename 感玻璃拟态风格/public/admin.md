
# 安全帽检测系统 - 管理端API接口文档

## 接口分类说明

### 🔄 需要更新的接口模块

以下模块对应管理端页面功能，需要根据实际业务需求更新接口实现：

- **仪表盘（Dashboard）** - 系统概览、统计分析、活动日志
- **警告管理（Warnings）** - 警告列表、状态管理、处理流程
- **数据导出（Export）** - 数据导出、报告生成

### ✅ 可直接使用的模板接口

以下模块可直接沿用现有模板接口，无需修改：

- **系统设置（Settings）** - 检测参数配置、警告规则设置
- **用户管理（User Management）** - 用户增删改查、权限管理

---

## 🔄 需要更新的接口模块

### 1. 仪表盘（Dashboard）

#### 1.1 获取系统状态概览

**接口地址**: `GET /safehat/api/system-overview/`
**中文说明**: 获取系统整体运行状态概览，包括摄像头、用户、警告、检测等统计信息

**响应示例**:

```json
{
 "code": 200,
 "msg": null,
 "data": {
  "cameras": {
   "total": 3,
   "online": 1,
   "offline": 2
  },
  "users": {
   "total": 12
  },
  "warnings": {
   "total": 19,
   "pending": 5,
   "today": 19
  },
  "detections": {
   "today": 50,
   "accuracy": 95.8
  },
  "system": {
   "availability": 33.33
  }
 }
}
```

#### 1.2 获取统计分析数据

**接口地址**: `GET /safehat/api/statistics/`
**中文说明**: 获取系统统计分析数据，包括检测趋势、警告趋势、摄像头活动等图表数据

**请求参数**:

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| type | string | 否 | 图表类型（daily/weekly/monthly），默认daily |
| days | int | 否 | 统计天数，默认7天 |

**响应示例**:

```json
{
 "code": 200,
 "msg": null,
 "data": {
  "detection_trend": [
   {
    "date": "2025-08-15",
    "count": 0
   },
  ],
  "warning_trend": [
   {
    "date": "2025-08-15",
    "count": 0
   },
  ],
  "warning_by_level": [
   {
    "warning_level": "medium",
    "count": 8
   },
  ],
  "camera_activity": [
   {
    "camera__name": "安全帽检测摄像头9号",
    "count": 8
   },
  ],
  "date_range": {
   "start": "2025-08-15",
   "end": "2025-08-22"
  }
 }
}
```

#### 1.3 获取检测统计数据

**接口地址**: `GET /safehat/api/detection-stats/`
**中文说明**: 获取指定摄像头或全部摄像头的检测统计数据，包括检测总数、合规率等

**请求参数**:

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| camera_id | string | 否 | 摄像头ID |
| date_range | int | 否 | 统计天数，默认7天 |

**响应示例**:

```json
{
 "code": 200,
 "msg": null,
 "data": {
  "stats": {
   "total_detections": 50,
   "person_count": 0,
   "wearing_hat_count": 0,
   "no_hat_count": 0,
   "avg_confidence": 0.746,
   "compliance_rate": 0
  }
 }
}
```

#### 1.4 获取活动日志

**接口地址**: `GET /safehat/api/activity-logs/`
**中文说明**: 获取系统活动日志记录，包括用户操作、系统事件、警告处理等日志信息

**请求参数**:

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| page | int | 否 | 页码，默认1 |
| page_size | int | 否 | 每页数量，默认20 |
| type | string | 否 | 日志类型（user/warning/system） |

**响应示例**:

```json

```

### 2. 警告管理（Warnings）

#### 2.1 获取警告列表

**接口地址**: `GET /safehat/api/warnings/`
**中文说明**: 获取系统中的安全警告列表，支持按状态、摄像头、严重程度等条件过滤

**请求参数**:

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| page | int | 否 | 页码，默认1 |
| page_size | int | 否 | 每页数量，默认10 |
| status | string | 否 | 状态过滤（pending/processing/resolved/ignored） |
| camera_id | string | 否 | 摄像头ID |
| severity | string | 否 | 严重程度（info/warning/critical） |
| date_from | string | 否 | 开始日期（YYYY-MM-DD） |
| date_to | string | 否 | 结束日期（YYYY-MM-DD） |

**响应示例**:

```json
{
 "code": 200,
 "msg": null,
 "data": {
  "total": 19,
  "page": 1,
  "page_size": 10,
  "warnings": [
   {
    "id": "20",
    "title": "未佩戴安全帽违规",
    "description": "在机械停放区检测到安全违规行为，需要立即处理。",
    "warning_level": "high",
    "status": "resolved",
    "camera_name": "安全帽检测摄像头8号",
    "created_at": "2025-08-22T11:01:09.749800",
    "resolved_by": null,
    "resolved_at": null
   }
  ]
 }
}
```

#### 2.2 更新警告状态

**接口地址**: `POST /safehat/api/warnings/update-status/`
**中文说明**: 更新指定警告的处理状态，如标记为已解决、忽略等

**请求参数**:

```json
{
  "warning_id": "123",
  "status": "resolved",
  "notes": "已处理完成"
}
```

**响应示例**:

```json
{
  "code": 200,
  "msg": "警告状态更新成功"
}
```

#### 2.3 删除警告

**接口地址**: `DELETE /safehat/api/warnings/<warning_id>/`
**中文说明**: 删除指定的安全警告记录

**路径参数**:

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| warning_id | string | 是 | 警告ID |

**响应示例**:

```json
{
  "code": 200,
  "msg": "警告删除成功"
}
```

#### 2.4 手动创建警告

**接口地址**: `POST /safehat/api/warnings/create/`
**中文说明**: 手动创建安全警告记录

**请求参数**:

```json
{
  "camera_id": "cam001",
  "warning_level": "high",
  "title": "安全帽违规",
  "description": "检测到未佩戴安全帽",
  "assigned_to": "username"
}
```

**响应示例**:

```json
{
  "code": 200,
  "msg": "警告创建成功",
  "data": {
    "warning_id": "456",
    "created_at": "2024-01-15T10:30:00Z"
  }
}
```

### 3. 数据导出（Export）

#### 3.1 导出数据

**接口地址**: `POST /safehat/api/data-export/`
**中文说明**: 导出指定类型和时间范围的数据，支持用户、警告、检测记录等数据导出

**请求参数**:

```json
{
  "type": "warnings",
  "start_date": "2024-01-01",
  "end_date": "2024-01-31"
}
```

**支持的导出类型**:

- `users`: 用户数据
- `warnings`: 警告数据
- `detections`: 检测记录

**响应示例**:

```json
{
  "code": 200,
  "msg": "success",
  "data": {
    "export_data": [
      {
        "id": "123",
        "camera_id": "CAM001",
        "warning_level": "warning",
        "status": "resolved",
        "title": "未佩戴安全帽警告",
        "description": "检测到人员未佩戴安全帽",
        "created_at": "2024-01-15 10:30:00"
      }
    ],
    "total_count": 1,
    "export_type": "warnings"
  }
}
```

---

## ✅ 可直接使用的模板接口

### 4. 摄像头管理（Camera Management）

#### 4.1 获取摄像头列表

**接口地址**: `GET /safehat/api/cameras/`
**中文说明**: 获取系统中所有摄像头的列表信息，支持分页查询和状态过滤

**请求参数**:

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| query | string | 否 | 搜索关键词（摄像头名称或ID） |
| page | int | 否 | 页码，默认1 |
| page_size | int | 否 | 每页数量，默认10 |
| status | string | 否 | 状态过滤（online/offline） |

**响应示例**:

```json
{
 "code": 200,
 "msg": null,
 "data": {
  "total": 3,
  "page": 1,
  "page_size": 10,
  "cameras": [
   {
    "camera_id": "SH009",
    "name": "安全帽检测摄像头9号",
    "location": "危险品存放区",
    "is_online": false,
    "ip_address": "192.168.2.108",
    "port": 1935,
    "rtsp_url": "rtsp://192.168.2.108:554/stream",
    "is_active": true,
    "last_heartbeat": "2025-08-22T10:49:05.960019",
    "created_at": "2025-08-22T10:57:05.960019"
   },
  ]
 }
}
```

### 5. 系统设置（Settings）

**说明**: 系统设置功能可直接使用现有模板接口，包括：

- 检测参数配置
- 警告规则设置
- 系统参数管理

### 6. 用户管理（User Management）

**说明**: 用户管理功能可直接使用现有模板接口，包括：

- 用户增删改查
- 角色权限管理
- 用户状态管理

---

## 接口开发优先级

### 高优先级（需要立即开发）

1. **仪表盘接口** - 系统概览、统计分析
2. **警告管理接口** - 警告列表、状态管理

### 中优先级（后续开发）

3. **数据导出接口** - 数据导出功能
4. **活动日志接口** - 日志记录查询

### 低优先级（可延后）

5. **摄像头管理接口优化** - 根据实际需求调整
6. **系统设置接口定制** - 根据业务需求定制化
