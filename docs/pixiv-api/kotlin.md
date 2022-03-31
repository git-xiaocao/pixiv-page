# 储存库

<a href="https://github.com/xiao-cao-x/pixiv-kotlin-api" target="_blank">
    <img src="https://github-readme-stats.vercel.app/api/pin/?username=xiao-cao-x&repo=pixiv-kotlin-api&theme=omni" 
        alt="pixiv-kotlin-api">
</a>

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