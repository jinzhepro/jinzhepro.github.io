之前做的一个expo项目里有用到google登陆功能，特此记录一下：

### 1.申请google credentials

首先需要登陆[google cloud](https://console.cloud.google.com/apis/credentials)申请Web ClientID、iOS ClientID、android ClientID。

#### Web ClientID
填入对应的后台地址，如果使用expo Go 调试还需要加入`auth.expo.io`相关url。

![Image](https://github.com/user-attachments/assets/67ea83a0-dd5f-448e-b177-241c40877e31)

> [!CAUTION]
> 经我测试expo相关url并没有用，可能是我配置错了，这部分可以使用expo development build进行调试。

#### iOS ClientID

填入iOS Bundle ID

![Image](https://github.com/user-attachments/assets/283506b4-985e-4883-bc3c-20bfc89ec3b9)

#### android ClientID

需要安装Java JDK，终端运行`cd ~/.android && keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android`，将生成SHA-1 fingerprint。

![Image](https://github.com/user-attachments/assets/27497754-ce1d-4b22-af26-526268fd86d5)

> [!WARNING]
> android ClientID我并没有使用，不确定是否正确。


### 2.使用

根据[文档](https://docs.expo.dev/versions/latest/sdk/auth-session/)安装相关组件

```bash
npx expo install expo-auth-session expo-crypto
```

在组建中使用：
```jsx
// 导入Google认证相关模块
import * as Google from 'expo-auth-session/providers/google'
// 导入WebBrowser模块用于处理认证会话
import * as WebBrowser from 'expo-web-browser'

// 处理可能存在的认证会话
WebBrowser.maybeCompleteAuthSession()

export default function LoginScreen(){
  // 使用Google认证hook，返回请求对象、响应对象和触发认证的函数
  const [request, response, promptAsync] = Google.useAuthRequest({
    webClientId: GOOGLE_CLIENT_ID,
    androidClientId: GOOGLE_ANDROID_CLIENT_ID,
    iosClientId: GOOGLE_IOS_CLIENT_ID,
  })

  // 监听认证响应
  useEffect(() => {
    // 当认证成功时处理返回的accessToken
    if (response?.type === 'success') {
      handleGoogleSignIn(response.authentication.accessToken)
    }
  }, [response])

  // 处理Google登录的函数
  const handleGoogleSignIn = async token => {
    if (!token) return

    try {
      // 调用后端API进行认证
      const { data, error } = await googleAuth({ access_token: token }).unwrap()

      // 处理错误情况
      if (error) {
        Toast.show({
          type: 'error',
          text2: error.message,
        })
      }

      // 认证成功，保存token并返回
      if (data) {
        dispatch(setToken(data.token))
        Toast.show({
          type: 'success',
          text2: '登录成功',
        })
        router.back()
      }
    } catch (error) {
      // 处理异常情况
      Toast.show({
        type: 'error',
        text2: '登录失败，请重试',
      })
    }
  }

  // 渲染Google登录按钮
  return (
    <TouchableOpacity
      className="items-center justify-center w-12 h-12 rounded-full bg-gray-100"
      onPress={async () => {
        await promptAsync() // 执行google Oauth
      }}
      disabled={loading}
    >
      <Icons.AntDesign name="google" size={24} color="#4285F4" />
    </TouchableOpacity>
  )
}
```