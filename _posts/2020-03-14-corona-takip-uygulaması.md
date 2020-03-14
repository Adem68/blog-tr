---
layout: post
title: Covidlock - Coronavirus İzleme Uygulaması Analizi
tags: ["corona", "virus", "mobile", "malware", "zararlı yazılım", "android", "uygulama", "covidlock"]
---

Merhaba,

Uzun bir aradan sonra yeni bir blog yazısıyla karşınızdayım.

## Giriş

Corona (nCoV-19, 2019-nCoV, korona) virüsü tüm dünyanın gündeminde olduğu için, zararlı yazılım geliştiren kötü niyetli hackerlar da hedef olarak coronayı seçmişler. coronavirusapp[.]site diye website açıp, apk'yı da site üzerinden yayınlamışlar. Girişi okuduğuna göre artık rivörse geçebiliriz.

## Reverse Time

Şimdi jadx-gui'yi açıp apk dosyamızı seçelim. Sonrasında MainActivity'nin nerede olduğunu öğrenmek için AndroidManifest.xml'e girip kodlara bir bakalım.

```java
        <activity android:name="com.device.security.activities.MainActivity" android:excludeFromRecents="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
```
MainActivity'nin nerede olduğunu öğrendik, şimdi uygulamanın ekran görüntülerine bakalım.


<img src="assets/img/corona-landing.png" alt="coronavirus app landing" border="0">

Güya bizi böyle bir arayüz karşılıyor ama gerçek arayüze bakalım 😏

<img src="assets/img/corona-main.png" alt="coronavirus app landing" border="0" copyright="DomainTools">




## Kaynaklar

https://www.domaintools.com/resources/blog/covidlock-mobile-coronavirus-tracking-app-coughs-up-ransomware (Bu blog yazısını yazmama sebep oldu, bu yazıda gördüğünüz resimler de bu siteden alındı)
