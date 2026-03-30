# Al-vpn
Hem yapay zeka hem VPN
AI VPN ANDROID SOURCE CODE
==========================

FILE: VpnService
--------------------------
package com.example.aivpn

import android.net.VpnService
import android.os.ParcelFileDescriptor
import android.util.Log
import java.io.FileInputStream
import java.io.FileOutputStream
import java.net.InetSocketAddress
import java.nio.ByteBuffer
import java.nio.channels.DatagramChannel

class MyVpnService : VpnService() {
    private var vpnInterface: ParcelFileDescriptor? = null
    private var vpnThread: Thread? = null

    override fun onStartCommand(intent: android.content.Intent?, flags: Int, startId: Int): Int {
        val action = intent?.action
        if (action == "STOP") {
            stopVpn()
            return START_NOT_STICKY
        }

        // VPN Yapılandırması ve Başlatma
        val serverAddress = intent?.getStringExtra("SERVER_ADDRESS") ?: "1.1.1.1"
        val serverPort = intent?.getIntExtra("SERVER_PORT", 51820) ?: 51820

        startVpn(serverAddress, serverPort)
        return START_STICKY
    }

    private fun startVpn(serverAddress: String, serverPort: Int) {
        vpnThread = Thread {
            try {
                val builder = Builder()
                builder.setSession("AI-VPN Secure Tunnel")
                builder.addAddress("10.0.0.2", 24)
                builder.addDnsServer("8.8.8.8")
                builder.addRoute("0.0.0.0", 0)
                
                // AI-Plus Optimization: MTU Tuning
                builder.setMtu(1400)

                vpnInterface = builder.establish()
                
                Log.d("VPN", "VPN Interface established")
                runVpnLoop(serverAddress, serverPort)
            } catch (e: Exception) {
                Log.e("VPN", "Error starting VPN", e)
            }
        }.apply { start() }
    }

    private fun runVpnLoop(serverAddress: String, serverPort: Int) {
        // Tünel veri döngüsü mantığı burada yer alır
        // DatagramChannel üzerinden paket iletimi
    }

    private fun stopVpn() {
        vpnThread?.interrupt()
        vpnInterface?.close()
        vpnInterface = null
        stopSelf()
    }

    override fun onDestroy() {
        stopVpn()
        super.onDestroy()
    }
}


FILE: MainActivity
--------------------------
package com.example.aivpn

import android.content.Intent
import android.net.VpnService
import android.os.Bundle
import androidx.activity.ComponentActivity

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // VPN İzni Kontrolü
        val intent = VpnService.prepare(this)
        if (intent != null) {
            // İzin isteniyor
            startActivityForResult(intent, 0)
        } else {
            // Zaten izin verilmiş, VPN'i başlatabilirsin
            startVpnService()
        }
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        if (resultCode == RESULT_OK) {
            startVpnService()
        }
    }

    private fun startVpnService() {
        val intent = Intent(this, MyVpnService::class.java).apply {
            putExtra("SERVER_ADDRESS", "45.158.12.44")
            putExtra("SERVER_PORT", 51820)
        }
        startService(intent)
    }
}


FILE: AndroidManifest
--------------------------
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.aivpn">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name">

        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <service
            android:name=".MyVpnService"
            android:permission="android.permission.BIND_VPN_SERVICE"
            android:exported="false">
            <intent-filter>
                <action android:name="android.net.VpnService" />
            </intent-filter>
        </service>

    </application>
</manifest>


FILE: VpnConfig
--------------------------
data class VpnConfig(
    val serverAddress: String,
    val serverPort: Int,
    val publicKey: String,
    val privateKey: String,
    val dns: String = "8.8.8.8",
    val mtu: Int = 1420
)


FILE: WgConfigGenerator
--------------------------
package com.example.aivpn

import com.wireguard.config.Config
import com.wireguard.config.Interface
import com.wireguard.config.Peer
import com.wireguard.config.InetNetwork
import com.wireguard.config.InetEndpoint
import com.wireguard.crypto.KeyPair
import com.wireguard.crypto.PublicKey

class WgConfigGenerator {

    /**
     * Yeni bir WireGuard anahtar çifti oluşturur.
     */
    fun generateKeys(): KeyPair {
        return KeyPair()
    }

    /**
     * Verilen parametrelerle tam bir WireGuard yapılandırması oluşturur.
     */
    fun createConfig(
        clientPrivateKey: String,
        clientAddress: String,
        serverPublicKey: String,
        serverEndpoint: String,
        dns: String = "1.1.1.1"
    ): Config {
        // 1. Arayüz (Interface) Yapılandırması
        val wgInterface = Interface.Builder()
            .addAddress(InetNetwork.parse(clientAddress))
            .setPrivateKey(KeyPair(clientPrivateKey).privateKey)
            .addDnsServer(java.net.InetAddress.getByName(dns))
            .build()

        // 2. Eş (Peer) Yapılandırması
        val wgPeer = Peer.Builder()
            .addAllowedIp(InetNetwork.parse("0.0.0.0/0")) // Tüm trafiği yönlendir
            .setEndpoint(InetEndpoint.parse(serverEndpoint))
            .setPublicKey(PublicKey.fromBase64(serverPublicKey))
            .setPersistentKeepalive(25)
            .build()

        // 3. Nihai Yapılandırma
        return Config.Builder()
            .setInterface(wgInterface)
            .addPeer(wgPeer)
            .build()
    }
}


FILE: BuildGradle
--------------------------
// build.gradle (Module: app)
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
}

android {
    namespace 'com.example.aivpn'
    compileSdk 34

    defaultConfig {
        applicationId "com.example.aivpn"
        minSdk 26
        targetSdk 34
        versionCode 1
        versionName "1.0"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = '1.8'
    }
}

dependencies {
    implementation 'androidx.core:core-ktx:1.12.0'
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'com.google.android.material:material:1.11.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    
    // WireGuard Android SDK
    implementation "com.wireguard.android:tunnel:1.0.20230706"
    implementation "com.wireguard.config:config:1.0.20230706"
}


FILE: SetupGuide
--------------------------
### AI VPN Android Kurulum Rehberi

1. **Android Studio'yu Açın:** Yeni bir "Empty Views Activity" projesi oluşturun.
2. **Paket Adı:** Proje paket adını `com.example.aivpn` olarak ayarlayın.
3. **Bağımlılıkları Ekleyin:** `build.gradle (app)` dosyasını açın ve paylaştığım bağımlılıkları (WireGuard SDK) ekleyip "Sync Now" yapın.
4. **Dosyaları Oluşturun:**
   - `MyVpnService.kt`
   - `MainActivity.kt`
   - `WgConfigGenerator.kt`
   - `VpnModule.kt` (React Native Köprüsü için)
   dosyalarını oluşturun ve kodları içine yapıştırın.
5. **Manifest:** `AndroidManifest.xml` dosyasını güncelleyin (Servis deklarasyonunu eklemeyi unutmayın).
6. **Cihazı Bağlayın:** Telefonunuzu USB ile bağlayın veya bir Emulator başlatın.
7. **Çalıştır:** "Run" butonuna basın. Uygulama telefonunuza yüklenecektir.

**Not:** VPN'in çalışması için telefonunuzda VPN izni vermeniz gerekecektir.


FILE: VpnModule
--------------------------
package com.example.aivpn

import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.bridge.ReactContextBaseJavaModule
import com.facebook.react.bridge.ReactMethod
import android.content.Intent

class VpnModule(reactContext: ReactApplicationContext) : ReactContextBaseJavaModule(reactContext) {
    override fun getName(): String = "AiVpnManager"

    @ReactMethod
    fun startVpn(serverIp: String, port: Int) {
        val intent = Intent(reactApplicationContext, MyVpnService::class.java).apply {
            putExtra("SERVER_ADDRESS", serverIp)
            putExtra("SERVER_PORT", port)
            action = "START"
        }
        reactApplicationContext.startService(intent)
    }

    @ReactMethod
    fun stopVpn() {
        val intent = Intent(reactApplicationContext, MyVpnService::class.java).apply {
            action = "STOP"
        }
        reactApplicationContext.startService(intent)
    }
}


FILE: ReactNative
--------------------------
import { NativeModules } from 'react-native';
const { AiVpnManager } = NativeModules;

// Arayüzdeki butona basıldığında:
const handleConnect = () => {
  // AI'dan gelen veriyi buraya basıyoruz
  AiVpnManager.startVpn("45.158.12.44", 51820); 
};

// Durdurmak için:
const handleDisconnect = () => {
  AiVpnManager.stopVpn();
};


FILE: FastAPI
--------------------------
# FastAPI (Backend) tarafında abonelik kontrolü
from fastapi import FastAPI, Depends
import database # Örnek veritabanı modülü

app = FastAPI()

@app.get("/check-subscription/{user_id}")
def check_sub(user_id: str):
    # Veritabanından (örneğin MongoDB veya PostgreSQL) kullanıcıyı sorgula
    user = database.users.find_one({"id": user_id})
    if not user or not user.get("is_premium"):
        return {
            "status": "free", 
            "servers": ["Limited-TR", "Limited-DE"],
            "message": "Upgrade to AI-Plus for full access"
        }
    
    return {
        "status": "premium", 
        "servers": "All-Access-AI",
        "features": ["Neural Routing", "Deep Shield"]
    }

<img width="1080" height="2340" alt="15240" src="https://github.com/user-attachments/assets/03b65029-dd7d-4f29-8385-4d533c4e1c7c" />
<img width="1080" height="2340" alt="15238" src="https://github.com/user-attachments/assets/b107ee8f-b94d-4005-8c1d-8b679f646be1" />
<img width="1080" height="2340" alt="15236" src="https://github.com/user-attachments/assets/5e1d9bde-257d-4381-83f9-83d80c4a5105" />
<img width="1080" height="2340" alt="feab823d-251f-4137-a75b-aba6bc19c77c-1_all_1394" src="https://github.com/user-attachments/assets/d55fd5e4-d55b-4a02-9c8f-18d6694afcf0" />
<img width="1080" height="2340" alt="15221" src="https://github.com/user-attachments/assets/1303ffff-9200-48b9-a029-b76d2384af9f" />
<img width="1080" height="2340" alt="15216" src="https://github.com/user-attachments/assets/da92cd04-8c93-43a4-bea0-1f35827baa35" />
