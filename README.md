# vladpadikbuildscript {

    repositories {
    
        jcenter()
        
        maven { 
        
            url = 'http://files.minecraftforge.net/maven'
        }
        
        maven {

            url 'https://plugins.gradle.org/m2/'
        }
    }
    
    dependencies {
    
        classpath 'net.minecraftforge.gradle:ForgeGradle:2.3-SNAPSHOT'
    }
}

plugins {

   id 'com.matthewprenger.cursegradle' version '1.1.0'
}

apply plugin: 'net.minecraftforge.gradle.forge'
apply plugin: 'com.matthewprenger.cursegradle'
apply plugin: 'maven-publish' 

apply from: 'https://raw.githubusercontent.com/MinecraftModDevelopment/Gradle-Collection/master/generic/markdown-git-changelog.gradle'
apply from: 'https://raw.githubusercontent.com/MinecraftModDevelopment/Gradle-Collection/master/minecraft/artifacts.gradle'
apply from: 'https://raw.githubusercontent.com/MinecraftModDevelopment/Gradle-Collection/master/minecraft/maven.gradle'

version = "${mod_version}" + getBuildNumber()
group = "${mod_group}"
archivesBaseName = "${mod_name}-${version_minecraft}"

sourceCompatibility = 1.8
targetCompatibility = 1.8

minecraft {

    version = "${version_minecraft}-${version_forge}"
    mappings = "${version_mcp}"
    runDir = 'run'
    
    replace '@VERSION@', project.version
    replace '@FINGERPRINT@', project.findProperty('signSHA1')
    replaceIn "${mod_class}.java"
}

repositories {

    maven {

        url "http://maven.mcmoddev.com"
    }
    
    maven {
    
        url 'http://maven.blamejared.com'
    }
}

dependencies {

    deobfCompile "net.darkhax.bookshelf:Bookshelf-1.12.2:${version_bookshelf}"
    deobfCompile "CraftTweaker2:CraftTweaker2-MC1120-Main:1.12-${version_minetweaker}"
}

processResources {

    inputs.property "version", project.version
    inputs.property "mcversion", project.minecraft.version

    from(sourceSets.main.resources.srcDirs) {
    
        include 'mcmod.info'
        expand 'version':project.version, 'mcversion':project.minecraft.version
    }
        
    from(sourceSets.main.resources.srcDirs) {
    
        exclude 'mcmod.info'
    }
    
    from 'LICENSE'
}

String getBuildNumber() {

    return System.getenv('BUILD_NUMBER') ? System.getenv('BUILD_NUMBER') : System.getenv('TRAVIS_BUILD_NUMBER') ? System.getenv('TRAVIS_BUILD_NUMBER') : '0';
}

//Shuts up javadoc failures
if (JavaVersion.current().isJava8Compatible()) {

    allprojects {

        tasks.withType(Javadoc) {

            options.addStringOption('Xdoclint:none', '-quiet')
        }
    }
}

curseforge {

    apiKey = System.getenv('curse_auth') ? System.getenv('curse_auth') : 0 
    def versions = "${curse_versions}".split(', ')

    project {

        id = "${curse_project}"
        releaseType = 'alpha'
        changelog = getGitChangelog
        changelogType = 'markdown'

        versions.each {

            addGameVersion "${it}"
        }

        if (project.hasProperty('curse_requirements') || project.hasProperty('curse_optionals')) {

            mainArtifact(jar) {

                relations {

                    if (project.hasProperty('curse_requirements')) {
                        def requirements = "${curse_requirements}".split(', ')
                        requirements.each {

                            requiredLibrary "${it}"
                        }
                    }

                    if (project.hasProperty('curse_optionals')) {
                        def optionals = "${curse_optionals}".split(', ')
                        optionals.each {

                            optionalLibrary "${it}"
                        }
                    }
                }
            }
        }

        addArtifact(sourcesJar)
        addArtifact(javadocJar)
        addArtifact(deobfJar)
    }
}

task signJar(type: SignJar, dependsOn: reobfJar) {

    onlyIf {
    
        project.hasProperty('keyStore')
    }
    
    keyStore = project.findProperty('keyStore')
    alias = project.findProperty('keyStoreAlias')
    storePass = project.findProperty('keyStorePass')
    keyPass = project.findProperty('keyStoreKeyPass')
    inputFile = jar.archivePath
    outputFile = jar.archivePath
}

build.dependsOn signJar

repositories {

    maven { url '' }
}

dependencies {

    compile "net.darkhax.gamestages:GameStages-${version_minecraft}:${version_gamestages}"
}
