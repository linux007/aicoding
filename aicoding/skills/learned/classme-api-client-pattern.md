# ClassMe API Client Implementation Pattern

**Extracted:** 2026-04-01
**Context:** When implementing new API clients in the classme codebase

## Problem
When creating new API clients in `api/` package, incorrectly using direct `base.ApiClient` method calls instead of the wrapper functions from `git.zuoyebang.cc/jxzt/classme/api`.

## Solution
Always use the HTTP utility wrappers from the `api` package (`api.HttpGet`, `api.HttpPostJson`, `api.HttpPostJosnByUnmarshal`, etc.) instead of calling `conf.API.<Service>.HttpGet()` directly.

Correct pattern:
```go
res, err := api.HttpGet(ctx, &conf.API.Genke, headers, queryParams)
```

Incorrect pattern (avoid):
```go
res, err := conf.API.Genke.HttpGet(ctx, pathInfo, opt)  // Direct call - WRONG
```

## Example
```go
// In api/genke/genke.go
func GetGenkeUrl(ctx *gin.Context, courseId, lessonId, deviceUid int64) (string, error) {
    headers := map[string]string{
        "pathinfo": "/genke/api/entry-link",
    }
    queryParams := map[string]string{
        "courseId":  fmt.Sprintf("%d", courseId),
        "lessonId":  fmt.Sprintf("%d", lessonId),
        "deviceUid": fmt.Sprintf("%d", deviceUid),
    }

    res, err := api.HttpGet(ctx, &conf.API.Genke, headers, queryParams)
    if err != nil {
        zlog.Warnf(ctx, "GetGenkeUrl failed, courseId=%d, err=%v", courseId, err)
        return "", err
    }

    respData, ok := res.(map[string]interface{})
    if !ok {
        return "", fmt.Errorf("unexpected response type")
    }

    link, ok := respData["link"].(string)
    if !ok || link == "" {
        return "", nil
    }
    return link, nil
}
```

## When to Use
- Creating new API client files in `api/<servicename>/` directories
- Adding HTTP calls to any service in the classme project
- Any new integration with internal services (achilles, tethys, roomcore, genke, etc.)

## Why This Matters
- The `api` package wrappers handle error wrapping, logging, metrics, and response parsing consistently
- Direct `base.ApiClient` calls bypass these abstractions and can cause type assertion errors (e.g., `*base.ApiResult` vs `interface{}`)
- Using wrappers matches the pattern established by all existing API clients in the codebase
