# Getting Started with Android RUM Collection

Datadog Real User Monitoring (RUM) enables you to visualize and analyze the real-time performance and user journeys of your application's individual users.

## Setup


1. Declare [dd-sdk-android][1] as a dependency in your `build.gradle` file:

    ```conf
    repositories {
        maven { url "https://dl.bintray.com/datadog/datadog-maven" }
    }

    dependencies {
        implementation "com.datadoghq:dd-sdk-android:x.x.x"
    }
    ```

2. [Specify application details in Datadog UI][2] to generate a unique Datadog Application ID, and Client Token.

{{< img src="docs/images/screenshot_rum.png" alt="RUM Event hierarchy" style="width:50%;border:none" >}}

To ensure safety of your data, you must use a client token: you cannot use [Datadog API keys][3] to configure the `dd-sdk-android` library as they would be exposed client-side in the Android application APK byte code. For more information about setting up a client token, see the [client token documentation][4]

3. Initialize the library with application context and start sending data:

    {{< tabs >}}
    {{% tab "US" %}}

   
```kotlin
   class SampleApplication : Application() {
       override fun onCreate() {
           super.onCreate()

          val configuration = Configuration.Builder()
                        .trackInteractions()
                        .trackLongTasks(durationThresold)
                        .useViewTrackingStrategy(strategy)
                        .build()
          val credentials = Credentials(<CLIENT_TOKEN>,<ENV_NAME>,<APP_VARIANT_NAME>,<APPLICATION_ID>)
          Datadog.initialize(this, credentials, configuration, trackingConsent)

       }
   }
```

    {{% /tab %}}
    {{% tab "EU" %}}
```kotlin
class SampleApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        val configuration = Configuration.Builder()
                        .trackInteractions()
                        .trackLongTasks(durationThresold)
                        .useViewTrackingStrategy(strategy)
                        .useEUEndpoints()
                        .build()
        val credentials = Credentials(<CLIENT_TOKEN>,<ENV_NAME>,<APP_VARIANT_NAME>,<APPLICATION_ID>)
        Datadog.initialize(this, credentials, configuration, trackingConsent)
          
    }
}
```
    {{% /tab %}}
    {{< /tabs >}}

Learn more about [ViewTrackingStrategy][5] to enable auto tracking of all your views (activities, fragments etc.), [trackingConsent][6] to add GDPR compliance for your EU users and [other configuration options][7] to initialize the library.

Note that in the credentials required for initialization, your application variant name is also required. This is important because it enables  the right proguard `mapping.txt` file to be automatically uploaded at build time to be able to view de-obfuscated stack traces.

4. Configure and register the RUM Monitor. You only need to do it once, usually in your application's `onCreate()` method:

    ```kotlin
    val monitor = RumMonitor.Builder()
            .build()
    GlobalRum.registerIfAbsent(monitor)
    ```


5. To track your OkHttp requests as resources, add the provided [Interceptor][8]:

    ```kotlin
    val okHttpClient =  OkHttpClient.Builder()
        .addInterceptor(DatadogInterceptor())
        .build()
    ```

    This records each request processed by the OkHttpClient as a Resource in RUM, with all the relevant information automatically filled (URL, method, status code, error). Note that only network requests started when a view is active will be tracked. If you want to track requests when your application is in the background, you can [create a view manually][9].

    **Note**: If you also use multiple Interceptors, this one must be called first.

You can further add an EventListener for the OkHttpClient to [automatically track resource timing][10] (3rd party providers, network requests). 


## Further Reading

{{< partial name="whats-next/whats-next.html" >}}

[1]: https://github.com/DataDog/dd-sdk-android
[2]: https://app.datadoghq.com/rum/create
[3]: https://docs.datadoghq.com/account_management/api-app-keys/#api-keys
[4]: https://docs.datadoghq.com/account_management/api-app-keys/#client-tokens
[5]: /real_user_monitoring/android/configure_android_sdk/track_view
[6]: /real_user_monitoring/android/troubleshooting_android/tracking_consent
[7]: /real_user_monitoring/android/configure_android_sdk/initialization_parameters
[8]: https://square.github.io/okhttp/interceptors/
[9]: https://square.github.io/okhttp/events/
[10]: /real_user_monitoring/android/configure_android_sdk/track_resource