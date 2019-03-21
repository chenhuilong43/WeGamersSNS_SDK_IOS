.. _topics-aar文件引用:

================
aar文件引用
================

在工程目录下生成目录文件aars，拷入aar文件Wegamers_SNS_SDK-release.aar

build.gradle文件下配置文件引用，导入aar需要的公共库

repositories {
    flatDir {
        dirs 'aars'
	
	}}

dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    
	compile 'com.android.support:appcompat-v7:23.4.0'
    
	//Retrofit
    
	compile ('com.squareup.retrofit2:converter-gson:2.2.0'){
        
		exclude group: 'com.squareup.okhttp3'
        
		exclude group: 'com.google.code.gson'
    
	}
    
	compile 'com.google.code.gson:gson:2.7'
    
	compile 'com.squareup.retrofit2:adapter-rxjava:2.2.0'
    
	compile 'com.squareup.okhttp3:logging-interceptor:3.6.0'
    
	//Rxjava
    
	compile 'io.reactivex:rxandroid:1.2.0'
    
	//Dagger
    
	compile 'com.google.dagger:dagger:2.2'
    
	compile 'org.greenrobot:eventbus:3.0.0'
    
	compile(name: 'Wegamers_SNS_SDK-release', ext: 'aar')
}
