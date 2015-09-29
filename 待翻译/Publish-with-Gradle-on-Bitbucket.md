使用Gradle将项目发布到Bitbucket上
---

> * 原文链接 : [Publish with Gradle on Bitbucket](https://medium.com/@Mul0w/publish-with-gradle-on-bitbucket-1463236dc460)
* 原文作者 : [Stef](https://medium.com/@Mul0w)
* 译文出自 : [开发技术前线 www.devtf.cn。未经允许，不得转载!](http://www.devtf.cn)
* 译者 : [MiJack](https://github.com/mijack)
* 校对者: [MiJack](https://github.com/mijack)
* 状态 :  已完成


Hi there !
大家好！

I often see public library hosted on MavenCentral, JCenter or others* … but what do you do for private things such as company libs, tools, etc ?

Of course you can see a few Maven private repository (ie. Nexus, Archiva), but in some cases, you don’t have hands on your company infrastructure. Assume that you’re an Android developer in a PHP driven company, with lots of processes … we can say that you’re almost screwed ^^

Almost ? Yes, because if you use a Git SCM like Github or Bitbucket, you can easily turn it as a Maven repository. Great, no ?

*Chris Banes already wrote a great article on Maven deployment with Gradle.

Let’s show some code to see how to do it !

Publish to a repo

Note : if you only use a single build.grade file, you can set everything in it. Or you can write a separate file to handle your deployments & call it with:

apply from: '<path-to-your-file>'
First, let’s apply the Maven plugin:

apply plugin: 'maven'
Then, let’s configure it: (sample here with Bitbucket, all the uppercase vars are properties from your project)

uploadArchives {
    configuration = configurations.archives
    repositories.mavenDeployer {
        pom.groupId = GROUP
        pom.artifactId = POM_ARTIFACT_ID
        pom.version = VERSION_NAME
        configuration = configurations.deployerJar
        repository(url: "git:releases://git@bitbucket.org:<bitbucket-username>/<your-repo>.git")
        snapshotRepository(url: "git:snapshots://git@bitbucket.org:<bitbucket-username>/<your-repo>.git")
        pom.project {
            name POM_NAME
            packaging POM_PACKAGING
            description POM_DESCRIPTION
            url POM_URL
            scm {
                url POM_SCM_URL
                connection POM_SCM_CONNECTION
                developerConnection POM_SCM_DEV_CONNECTION
            }
            licenses {
                license {
                    name POM_LICENCE_NAME
                    url POM_LICENCE_URL
                    distribution POM_LICENCE_DIST
                }
            }
            developers {
                developer {
                    id POM_DEVELOPER_ID
                    name POM_DEVELOPER_NAME
                    email POM_DEVELOPER_EMAIL
                }
            }
        }
    }
}
Well, here you find the deployerJar configuration, we need to set this up.

First, we add the Synergian Wagon-Git dependency:

allprojects {
    repositories {
        mavenCentral()
        maven { url "https://raw.github.com/synergian/wagon-git/releases"}
    }
}
&

dependencies {
    deployerJar "ar.com.synergian:wagon-git:0.2.3"
}
and the last step is to declare the configuration:

configurations { 
    deployerJar
}
Now you can deploy:

$ gradle [clean build] uploadArchives
And … you’re done !

Fetch your lib !
Well, now that you are able to publish your libs (private or not) to a repo hosted on Bitbucket (or others), it’s pretty easy to access it:

allprojects {
    repositories {
        mavenCentral()
        maven {
            url "https://api.bitbucket.org/1.0/repositories/<bitbucket_username>/<bitbucket-repository>/raw/snapshots"
            credentials {
                username REPOSITORY_USERNAME
                password REPOSITORY_PASSWORD
            }
        }
        maven {
            url "https://api.bitbucket.org/1.0/repositories/<bitbucket_username>/<bitbucket-repository>/raw/releases"
            credentials {
                username REPOSITORY_USERNAME
                password REPOSITORY_PASSWORD
            }
        }
    }
}
Of course, it’s better not to set your credentials in your build.gradle project file which is a versioned file, but you can set them in your home gradle.properties settings. One of the best way to be “secure” would be to have a dedicated user to fetch your libs, with a read-only profile, to whom your repository is shared … By the way, notice that the credentials are only necessary if you use a private repository.

And then, finally, just add your new dependency, just as usual:

dependencies {
      compile "GROUP-ID:ARTIFACT-ID:VERSION"
}
You can find the deploy.gradle file on github

That’s it ;-)