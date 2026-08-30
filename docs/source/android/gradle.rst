gradle
========

1. android studio初始化相关文件。

#. 手动添加国内镜像。

#. 再用cursor辅助开发。


gradle.properties
-------------------

.. code-block::

        systemProp.http.proxyHost=127.0.0.1
        systemProp.http.proxyPort=8080
        systemProp.https.proxyHost=127.0.0.1
        systemProp.https.proxyPort=8080


gradle-wrapper.properties 
-----------------------------

仅当前 project，在gradle/wrapper/gradle-wrapper.properties 文件中修改

.. code-block::

        distributionUrl=https\://mirrors.cloud.tencent.com/gradle/gradle-8.13-bin.zip


全局project，新建文件 ~/.gradle/init.d/gradle-mirror.gradle ，内容例如

.. code-block::

        allprojects {
            gradle.beforeSettings { settings ->
                def wrapper = settings.gradle.startParameter.gradleUserHomeDir.toPath()
                def wrapperProps = settings.settingsDir.toPath().resolve("gradle/wrapper/gradle-wrapper.properties").toFile()
                if (wrapperProps.exists()) {
                    def text = wrapperProps.text
                    def aliyunUrl = "https://mirrors.cloud.tencent.com/gradle/"
                    if (text.contains("https://services.gradle.org/distributions/")) {
                        text = text.replace("https://services.gradle.org/distributions/", aliyunUrl)
                        wrapperProps.text = text
                        println "[init.d] use Aliyun mirror for: ${settings.rootProject.name}"
                    }
                }
            }
        }



settings.gradle
-----------------

.. code-block::

        pluginManagement {
            repositories {
                maven { url 'https://maven.aliyun.com/repository/google' }
                maven { url 'https://maven.aliyun.com/repository/central' }
                maven { url 'https://maven.aliyun.com/repository/gradle-plugin' }
                google()
                mavenCentral()
                gradlePluginPortal()
            }
        }

        dependencyResolutionManagement {
            repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
            repositories {
                maven { url 'https://maven.aliyun.com/repository/google' }
                maven { url 'https://maven.aliyun.com/repository/central' }
                maven { url 'https://maven.aliyun.com/repository/public' }
                google()
                mavenCentral()
            }
        }
        rootProject.name = "myappname"
        include(":app")


build.gradle
--------------

.. code-block::

        plugins {
            alias(libs.plugins.android.application) apply false
        }



app/build.gradle
--------------------

.. code-block::

        plugins {
            alias(libs.plugins.android.application)
        }

        android {
            namespace = 'com.example.myappcert'
            compileSdk 36

            defaultConfig {
                applicationId "com.example.myappcert"
                minSdk 28
                targetSdk 36
                versionCode 1
                versionName "1.0"

                testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
            }

            buildTypes {
                release {
                    minifyEnabled false
                    proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
                }
            }
            compileOptions {
                sourceCompatibility JavaVersion.VERSION_11
                targetCompatibility JavaVersion.VERSION_11
            }
        }

        dependencies {
            implementation libs.appcompat
            implementation libs.material
            implementation libs.okhttp
            implementation libs.jjwt.api
            runtimeOnly libs.jjwt.impl
            runtimeOnly libs.jjwt.jackson
            testImplementation libs.junit
            androidTestImplementation libs.ext.junit
            androidTestImplementation libs.espresso.core
        }


libs.versions.toml
------------------------

.. code-block::

        [versions]
        agp = "8.13.0"
        junit = "4.13.2"
        junitVersion = "1.1.5"
        espressoCore = "3.5.1"
        appcompat = "1.6.1"
        material = "1.10.0"
        okhttp = "4.12.0"
        jjwt = "0.12.3"

        [libraries]
        junit = { group = "junit", name = "junit", version.ref = "junit" }
        ext-junit = { group = "androidx.test.ext", name = "junit", version.ref = "junitVersion" }
        espresso-core = { group = "androidx.test.espresso", name = "espresso-core", version.ref = "espressoCore" }
        appcompat = { group = "androidx.appcompat", name = "appcompat", version.ref = "appcompat" }
        material = { group = "com.google.android.material", name = "material", version.ref = "material" }
        okhttp = { group = "com.squareup.okhttp3", name = "okhttp", version.ref = "okhttp" }
        jjwt-api = { group = "io.jsonwebtoken", name = "jjwt-api", version.ref = "jjwt" }
        jjwt-impl = { group = "io.jsonwebtoken", name = "jjwt-impl", version.ref = "jjwt" }
        jjwt-jackson = { group = "io.jsonwebtoken", name = "jjwt-jackson", version.ref = "jjwt" }

        [plugins]
        android-application = { id = "com.android.application", version.ref = "agp" }


