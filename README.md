# Upload-to-bintray
### 
一、项目根build.gradle dependencies 添加
```xml
        classpath 'com.github.dcendents:android-maven-gradle-plugin:1.5'
        classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.8.4'
        classpath "org.jfrog.buildinfo:build-info-extractor-gradle:4.0.0" 
```

二、添加
```xml
ext {
    siteUrl = '
    gitUrl = ''
    group = ""
    version = ""
    id = ''
    name = ''
    email =  ''
    packaging = 'aar'

}
```

三、要上传的library添加
```xml
apply plugin: 'com.github.dcendents.android-maven'
apply plugin: 'com.jfrog.bintray'
def libname = ''
def libdesc = '' 
version = rootProject.ext.version
group = rootProject.ext.group

//生成源文件
task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
    classifier = 'sources'
}
//生成文档
task javadoc(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
    options.encoding "UTF-8"
    options.charSet 'UTF-8'
    options.author true
    options.version true
    failOnError false
}

//文档打包成jar
task javadocJar(type: Jar, dependsOn: javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}
//拷贝javadoc文件
task copyDoc(type: Copy) {
    from "${buildDir}/docs/"
    into "docs"
}

//上传到jcenter所需要的源码文件
artifacts {
    archives javadocJar
    archives sourcesJar
}

// 配置maven库，生成POM.xml文件
install {
    repositories.mavenInstaller {
        // This generates POM.xml with proper parameters
        pom {
            project {
                packaging rootProject.ext.packaging
                name libname
                url siteUrl
                licenses {
                    license {
                        name libname
                        url rootProject.ext.siteUrl
                    }
                }
                developers {
                    developer {
                        id rootProject.ext.id
                        name rootProject.ext.name
                        email rootProject.ext.email
                    }
                }
                scm {
                    connection rootProject.ext.gitUrl
                    developerConnection rootProject.ext.gitUrl
                    url rootProject.ext.siteUrl
                }
            }
        }
    }
}
Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())
bintray {
    user = properties.getProperty("bintray.user")    
    key = properties.getProperty("bintray.apikey")  
    configurations = ['archives']
    pkg {
        repo = "maven"
        name = libname  
        desc = libdesc  
        websiteUrl = rootProject.ext.siteUrl
        vcsUrl = rootProject.ext.gitUrl
        licenses = ["Apache-2.0"]
        publish = true
    }
}
```

四、local.properties 添加
```xml
bintray.user=xxxx
bintray.apikey=xxxx
```
