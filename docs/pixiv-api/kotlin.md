# 储存库

[![](https://github-readme-stats.vercel.app/api/pin/?username=git-xiaocao&repo=pixiv-kotlin-api&theme=omni)](https://github.com/git-xiaocao/pixiv-kotlin-api)

## 特点

1. Ktor Client
2. 刷新AuthToken支持

## 使用方法

```kotlin
val jsonFile = File("./userAccount.json")
val apiClient = ApiClient(Json.decodeFromString(jsonFile.readText())) {
    jsonFile.writeText(Json.encodeToString(it))
}
```