# [DEPRECATED]

안녕하세요! 아임포트 서비스를 이용해 주셔서 감사합니다!

이 프로젝트는 그동안 잘 사용됐으나, 지금은 ⛔ **Deprecated** ⛔  되었습니다.

앞으로는 🌱 [아임포트에서 제공하는 최신 SDK][123] 🌱 를 사용하셔서 

더 편리하고, 안정적인 결제 개발에 도움 받으시길 바랍니다!

[123]: https://github.com/iamport/iamport-android


# kcp-android-gradle
KCP안드로이드 결제연동 샘플입니다. 

## 인증앱으로 이동  
신용카드 결제 중 ISP결제, 실시간계좌이체를 위한 Bankpay결제를 위해서는 개발 중인 앱에서 인증용 앱(ISP, Bankpay)으로 이동이 필요합니다.  

외부 앱 호출/이동은 scheme을 통해 이뤄지게 되므로, intent호출이 정상적으로 이뤄질 수 있도록 `WebViewClient`가 지정되어야 합니다.  
*`KcpWebViewClient`는 구현되어 포함되어있으므로 import해 사용하면 됩니다.*    

```java
payWebView.setWebViewClient(new KcpWebViewClient(this, payWebView));
```

```java
public class KcpWebViewClient extends WebViewClient {

	@Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {

        if (!url.startsWith("http://") && !url.startsWith("https://") && !url.startsWith("javascript:")) {
            Intent intent = null;

            try {
                intent = Intent.parseUri(url, Intent.URI_INTENT_SCHEME); //IntentURI처리
                Uri uri = Uri.parse(intent.getDataString());

                activity.startActivity(new Intent(Intent.ACTION_VIEW, uri));
                return true;
            } catch (URISyntaxException ex) {
                return false;
            } catch (ActivityNotFoundException e) {
                if (intent == null) return false;

                if (handleNotFoundPaymentScheme(intent.getScheme())) return true;

                String packageName = intent.getPackage();
                if (packageName != null) {
                    activity.startActivity(new Intent(Intent.ACTION_VIEW, Uri.parse("market://details?id=" + packageName)));
                    return true;
                }

                return false;
            }
        }

        return false;
    }

}
```


## 인증앱으로부터 복귀  
ISP, Bankpay, PAYCO, 카드사앱으로부터 앱으로 복귀하면서 기존에 진행 중인 WebView프로세스가 연속되어 이어지게 됩니다.  

### 방법 1. scheme intent-filter 생략  

안드로이드 특성상, 외부 앱으로 빠져나갔다가 ISP / Bankpay / 카드사 앱카드 인증이 종료되면 인증앱의 Activity 종료만으로도 자동으로 원래의 앱으로 복귀하게 됩니다.  

때문에 scheme intent-filter를 굳이 선언하지 않아도 정상동작하게 됩니다. 

### 방법 2. android:launchMode="singleInstance"  

`IMP.request_pay(param, callback)`호출 시 param.app_scheme이 전달되었을 때 ISP앱의 경우 명시적으로 app_scheme을 호출하여 앱으로 복귀하려 시도하게 됩니다.  

ISP앱 인증 후 복귀했을 때, 기존에 결제를 진행 중이던 WebView로 돌아가 다음 작업을 이어가야하므로 새로운 Activity객체가 생성되지 않고 원래의 Activity로 복귀할 수 있도록 다음과 같이 설정합니다. 

- 결제 진행 중인 WebView가 포함된 Activity에 scheme에 대한 intent-filter를 선언
- android:launchMode="singleInstance" 속성을 선언

```xml
<intent-filter>
	<action android:name="android.intent.action.VIEW" />
	<category android:name="android.intent.category.DEFAULT" />
	<category android:name="android.intent.category.BROWSABLE" />
	<data android:scheme="iamporttest" />
</intent-filter>
```

