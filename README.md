# SSL Pinning android APK

SLL Pinning é uma técnica usada para quebrar o 
protocolo ssl e visualizar os dados de aplicativos.

## Frida no Android sem root

Para isso vai ser necessário injetarmos uma lib maliciosa do frida o frida-gadget dentro da apk a qual gostariamos de visualizar com o frida, para isso iremos fazer o seguinte:

Use o apktool para descompactar o APK em um diretório:

```bash
apktool d myapp.apk -o extractedFolder
```

Após isso iremos injetar uma lib [Frida Gadget](https://github.com/frida/frida/releases/) escolha a correta para arquitetura de seu sistema.


```
$ apktool --version
2.2.2

$ apktool d -o out_dir original.apk
I: Using Apktool 2.2.2 on original.apk
I: Loading resource table...
I: Decoding AndroidManifest.xml with resources...
I: Loading resource table from file: ~/.local/share/apktool/framework/1.apk
I: Regular manifest package...
I: Decoding file-resources...
I: Decoding values XMLs...
I: Baksmaling classes.dex...
I: Copying assets and libs...
I: Copying unknown files...
I: Copying original files...

# download frida gadget - for 32bit ARM in this case
$ wget https://github.com/frida/frida/releases/download/9.1.26/frida-gadget-9.1.26-android-arm.so.xz
2017-04-11 10:48:45 (3.29 MB/s) - ‘frida-gadget-9.1.26-android-arm.so.xz’ saved [3680748/3680748]

# extract the compressed archive
$ unxz frida-gadget-9.1.26-android-arm.so.xz

$ ls
frida-gadget-9.1.26-android-arm.so

# copy frida gadget library in armeabi directory under lib
$ cp frida_libs/armeabi/frida-gadget-9.1.26-android-arm.so out_dir/lib/armeabi/libfrida-gadget.so
```

Iremos chamar a nossa lib na primeira execução do programa, logo antes do return:

```
const-string v0, "frida-gadget"
invoke-static {v0}, Ljava/lang/System;->loadLibrary(Ljava/lang/String;)V
```
Se por um acaso o aplicativo não tiver acesso a internet, insira isso no Manifest:
```
<uses-permission android:name="android.permission.INTERNET" />
```
Após isso faça o build da aplicação novamente e assine com seu próprio certificado.
```
$ apktool b -o repackaged.apk out_dir/
I: Using Apktool 2.2.2
I: Checking whether sources has changed...
I: Smaling smali folder into classes.dex...
I: Checking whether resources has changed...
I: Building resources...
I: Copying libs... (/lib)
I: Building apk file...
I: Copying unknown files/dir...
```

Assinando o aplicativo:

```
# if you dont have a keystore already, here's how to create one
$ keytool -genkey -v -keystore custom.keystore -alias mykeyaliasname -keyalg RSA -keysize 2048 -validity 10000

# sign the APK
$ jarsigner -sigalg SHA1withRSA -digestalg SHA1 -keystore mycustom.keystore -storepass mystorepass repackaged.apk mykeyaliasname

# verify the signature you just created
$ jarsigner -verify repackaged.apk

# zipalign the APK and ApkSigner
$ zipalign.exe -p 4 base.apk base_aligned.apk
$ apksigner.exe -ks Store.ks base_aligned.apk
```

