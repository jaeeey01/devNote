# gradle 구성 하기
그래들은 우선 settings.gradle 파일을 참고해 프로젝트의 구조를 파악 <br>
그런 다음 개별 프로젝트를 build.gradle 을 통해 빌드하는 방식으로 동작

---

## settings.gradle
 전체 프로젝트의 구조 빌드

일반적으로 한개의 프로젝트 구성시
```groovy
rootProject.name="project-name"
include "project-name"
```

여러 모듈 프로젝트들을 포함하는 경우
```groovy
rootProject.name="project-name"
include ":sub-project1"
include ":sub-project2"
```


 모듈 프로젝트들이 많아서 이들을 group 으로 관리하고 싶다면 아래와 같은 스크립트 사용시 편리
```groovy
rootProject.name = 'security-gradle3'

["comp", "web", "server"].each {

    def compDir = new File(rootDir, it)
    if(!compDir.exists()){
        compDir.mkdirs()
    }

    compDir.eachDir {subDir ->

        def gradleFile = new File(subDir.absolutePath, "build.gradle")
        if(!gradleFile.exists()){
            gradleFile.text =
                    """

                    dependencies {

                    }

                    """.stripIndent(20)
        }

        [
                "src/main/java/com/sp/fc",
                "src/main/resources",
                "src/test/java/com/sp/fc",
                "src/test/resources"
        ].each {srcDir->
            def srcFolder = new File(subDir.absolutePath, srcDir)
            if(!srcFolder.exists()){
                srcFolder.mkdirs()
            }
        }

        def projectName = ":${it}-${subDir.name}";
        include projectName
        project(projectName).projectDir = subDir
    }
}
```

---

## build.gradle

루트 폴더의 build.gradle(이후 컴포넌트 별로 개별 build.gradle이 생성되므로 루트 폴더라고 표기 ) 에서는 
<br> 
전체 / 하위 프로젝트의 공통 설정에 대한 사항을 기술

```groovy

buildscript {
    ext {
        spring = "2.4.1"
        boot = "org.springframework.boot"
        lombok = "org.projectlombok:lombok"
    }
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("$boot:spring-boot-gradle-plugin:$spring")
    }
}

allprojects {
    group = "com.sp.fc"
    version = "1.0.0"
}

subprojects {

    apply plugin: "java"
    apply plugin: boot
    apply plugin: "io.spring.dependency-management"
    apply plugin: "idea"

    repositories {
        mavenCentral()
    }

    configurations {
        developmentOnly
        runtimeClasspath {
            extendsFrom developmentOnly
        }
    }

    dependencies {
        developmentOnly("$boot:spring-boot-devtools")
        implementation "$boot:spring-boot-starter-security"
        implementation 'com.fasterxml.jackson.core:jackson-annotations'

        compileOnly lombok
        testCompileOnly lombok
        annotationProcessor lombok
        testAnnotationProcessor lombok

        testImplementation "$boot:spring-boot-starter-test"
    }

    test {
        useJUnitPlatform()
    }

}


["comp", "web"].each {
    def subProjectDir = new File(projectDir, it)
    subProjectDir.eachDir {dir->
        def projectName = ":${it}-${dir.name}"
        project(projectName){
            //공통컴포넌트의 경우 bootJar이 불필요하기에 제거
            bootJar.enabled(false) 
            jar.enabled(true)
        }
    }
}
["server"].each {
    def subProjectDir = new File(projectDir, it)
    subProjectDir.eachDir {dir->
        def projectName = ":${it}-${dir.name}"
        project(projectName){

        }
    }
}

help.enabled(false)

```

---

## 프로젝트 auth reload 확성화 하기

- IntelliJ 에서 compiler.automake.allow.when.app.running 을 체크
- 설정의 Build project automatically 를 체크
- Run configuration 에서 On 'Update' action 과 On frame deactivation 의 값을 적절하게 수정

참고 사이트 : https://velog.io/@bread_dd/Spring-Boot-Devtools
