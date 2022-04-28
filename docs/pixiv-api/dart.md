# 储存库

[![](https://github-readme-stats.vercel.app/api/pin/?username=git-xiaocao&repo=pixiv_dart_api&theme=omni)](https://github.com/git-xiaocao/pixiv_dart_api)

[Pub地址](https://pub.dev/packages/pixiv_dart_api)

在`pubspec.yaml`添加`pixiv_dart_api`并刷新

```yaml
dependencies:
  pixiv_dart_api: latest
```

# 特点

1. 初始化和刷新AuthToken支持
2. 免代理IP直连
3. `GetxService`支持

# 使用方法

添加两个函数用来获取当前账号和更新当前账号

```dart
UserAccount currentAccount() {
  //获取当前账号
  return
  ...;
}

void updateAccount(UserAccount data) {
  //更新当前账号
  ...
}
```

## 直接使用

创建`PixivAuth`和`PixivApi`

```dart

final auth = PixivAuth(
  targetIPGetter: () => "210.140.92.183",
  languageGetter: () => "zh-CN",
  deviceName: "Redmi",
);

final api = PixivApi(
  auth: authClient,
  targetIPGetter: () => "210.140.92.183",
  languageGetter: () => "zh-CN",
  deviceName: "Redmi",
  accountGetter: () => currentAccount(),
  accountUpdater: (data) => updateAccount(data),
);

```

## GetX使用

给`PixivAuth`和`PixivApi`包装一层`GetxService`

```dart
class ApiClient extends GetxService with PixivApi {
  ApiClient initSuper(AuthClient authClient) {
    super.init(
      auth: authClient,
      targetIPGetter: () => "210.140.92.183",
      languageGetter: () => "zh-CN",
      deviceName: "Redmi",
      accountGetter: () => currentAccount(),
      accountUpdater: (data) => updateAccount(data),
    );
    return this;
  }
}

class AuthClient extends GetxService with PixivAuth {
  AuthClient initSuper() {
    super.init(
      targetIPGetter: () => "210.140.92.183",
      languageGetter: () => "zh-CN",
      deviceName: "Redmi",
    );
    return this;
  }
}

```

然后`Get.put`或者`Get.lazyPut`

```dart
void inject() {
  Get.lazyPut(() => AuthClient().initSuper());
  Get.lazyPut(() => ApiClient().initSuper(Get.find()));
}
```

# 常见问题

## UserAccount怎么获取?

需要WebView登录来获取

```dart

final AuthClient authClient = Get.find();
final s = authClient.randomString(128);
final url = 'https://app-api.pixiv.net/web/v1/' +
    (create ? 'provisional-accounts/create' : 'login') +
    '?code_challenge=${authClient.generateCodeChallenge(s)}&code_challenge_method=S256&client=pixiv-android';
```

WebView中添加以下代码

```kotlin
override fun shouldOverrideUrlLoading(
    view: WebView?,
    request: WebResourceRequest?
): Boolean {
    request?.let {
        val uri = it.url

        if ("pixiv" == uri.scheme) {

            val host = uri.host
            if (null != host && host.contains("account")) {

                try {
                    messageChannel.send(
                        mapOf(
                            "type" to "code",
                            "content" to uri.getQueryParameter("code")
                        )
                    )

                } catch (ignored: Exception) {
                }
            }
            return true
        }


    }
    return super.shouldOverrideUrlLoading(view, request)
}
```

Flutter中监听消息

```dart
  @override
void initState() {
  const BasicMessageChannel(
    '$_pluginName/result',
    StandardMessageCodec(),
  ).setMessageHandler(
        (dynamic message) async {
      final map = message as Map;
      switch (map['type']) {
        case 'code':
          final code = map['content'] as String;

          Log.i('code:$code');
          authClient.initAccountAuthToken(code, s).then((result) {
            // 到这里就登录成功了

            // final firstAccount = accountManager.isEmpty;

            //把账号储存到本地
            // accountManager.add(result);
            // Get.back();
            //因为登录了 所以不再显示导航页
            // appInfo.guideInit = true;

            // if (firstAccount) {
            //   Get.offAll(const AppBodyPage());
            // } else {
            //   Get.back();
            // }
          }).catchError((e) {
            Log.e('登录失败', e);
          });

          break;
      }
      return null;
    },
  );
}
```

## 怎么翻页

支持if判断的类型的翻页(Dart只能这样写)

```dart
  Future<T> next<T>(String url, {
  required CancelToken cancelToken,
}) async {
  final response = await _httpClient.get<String>(
    url.replaceFirst("app-api.pixiv.net", targetIPGetter.call()),
    cancelToken: cancelToken,
  );

  final responseData = response.data!;

  if (T == Comments) {
    return Comments.fromJson(jsonDecode(responseData)) as T;
  }
  if (T == Illusts) {
    return Illusts.fromJson(jsonDecode(responseData)) as T;
  }
  if (T == Novels) {
    return Novels.fromJson(jsonDecode(responseData)) as T;
  }
  if (T == Users) {
    return Users.fromJson(jsonDecode(responseData)) as T;
  }
  if (T == SearchIllust) {
    return SearchIllust.fromJson(jsonDecode(responseData)) as T;
  }
  if (T == BookmarkTags) {
    return BookmarkTags.fromJson(jsonDecode(responseData)) as T;
  }
  throw Exception('类型错误');
}
```

```dart
Feture<void> fetch() {
  final ApiClient api = Get.find();
  //获取第一页
  final result = await api.getRecommendedIllusts(IllustType.illust);
  
  //如果有下一页
  if (null != result.nextUrl) {
    final nextResult = api.next<Illusts>(url: result.nextUrl!);
  }
}
```

参考 [Pixiv Func Mobile](https://github.com/git-xiaocao/pixiv_func_mobile/blob/main/lib/pages/login/controller.dart#L30) 项目  
