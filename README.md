#MavenReference Proof-of-Concept

This project uses an MSBuild `IKVM.JavaReference.targets` file and a temporary NuGet package named `IKVM.JavaReference` to demonstrate how making a `<MavenReference>` project element can function. It uses the `mvn` command line utility to resolve the Maven dependencies and then uses the [copy-dependencies](https://maven.apache.org/plugins/maven-dependency-plugin/copy-dependencies-mojo.html) plugin to copy the `.jar` files from the `.m2` cache to the `obj/jar/` directory of the project.

There are 3 stages to getting this to work:

1. Resolve/download Maven Dependencies to `obj/jar/` directory
2. Execute `ikvmc` on the items in `obj/jar/` for each target framework
3. Self-copying the `.targets` file and `<MavenReference>`s recursively to each project that depends on us during `dotnet pack`

Only stage 1 is complete at this point. However, having a standalone `.targets` file with no dependencies makes it fairly straightforward to use.

The `IKVM.JavaReference.targets` file also attempts to use the `MAVEN_HOME` environment variable to make use of Maven if it is installed on the system. If not, it will download Maven and install it in the NuGet cache.

## Prerequisites

- OpenJDK 9 or higher
- `JAVA_HOME` must be configured to use a JDK (the directory name must include "jdk" in it)
- Visual Studio 2019 or higher

## Usage

After building `IKVM.JavaReference.csproj`, the NuGet package will be output to the `<root>/_artifacts/NuGetPackages/` directory. This can be used one of 2 ways:

- Setup a [local feed](https://docs.microsoft.com/en-us/nuget/hosting-packages/local-feeds) to this directory.
- Upload the `.nupkg` file to a NuGet artifact server.

Add a `<PackageReference>` to `IKVM.JavaReference` in a new project. Then add a `<MavenReference>` to any dependency found in https://search.maven.org/.

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netcoreapp3.1</TargetFramework>
    <!-- MavenHome can be set as a property or as MAVEN_HOME environment variable. -->
    <!-- It should not include /bin! -->
    <!--<MavenHome>F:\mvn-sample\apache-maven-3.8.5</MavenHome>-->
  </PropertyGroup>

  <ItemGroup>
    <!-- NOTE: This reference will end up in the .nuget cache, so whenever
      IKVM.JavaReference is updated, the .nuget cache must be manually deleted. -->
    <PackageReference Include="IKVM.JavaReference" Version="1.0.0" />
  </ItemGroup>
    
  <ItemGroup>
    <MavenReference Include="opennlp-tools" Version="1.9.1" GroupID ="org.apache.opennlp" />
    <MavenReference Include="opennlp-uima" Version="1.9.1" GroupID ="org.apache.opennlp" />
    <MavenReference Include="opennlp-morfologik-addon" Version="1.9.1" GroupID ="org.apache.opennlp" />
    <MavenReference Include="opennlp-brat-annotator" Version="1.9.1" GroupID ="org.apache.opennlp" />
  </ItemGroup>
</Project>
```

When you build the project, it will automatically resolve versions and download the Maven dependencies to `obj/jar/` of the current project.

```console
Rebuild started...
1>------ Rebuild All started: Project: MavenExample, Configuration: Debug Any CPU ------
Restored F:\Projects\_Demos\MavenReferencePoC\src\IKVM.JavaReference\IKVM.JavaReference.csproj (in 7 ms).
Restored F:\Projects\_Demos\MavenReferencePoC\src\MavenExample\MavenExample.csproj (in 21 ms).
1>You are using a preview version of .NET. See: https://aka.ms/dotnet-core-preview
1>Installing Maven...
1>[SUCCESS] Downloaded F:\Users\shad\.nuget\packages\ikvm.javareference\1.0.0\maven/maven.zip from https://dlcdn.apache.org/maven/maven-3/3.8.5/binaries/apache-maven-3.8.5-bin.zip
1>[SUCCESS] Maven Installed
1>MavenHome: F:\Users\shad\.nuget\packages\ikvm.javareference\1.0.0\maven/apache-maven-3.8.5
1>Writing POM file to 'F:\Projects\_Demos\MavenReferencePoC\src\MavenExample\obj\jar/pom.xml'.
1>[SUCCESS] Wrote 'F:\Projects\_Demos\MavenReferencePoC\src\MavenExample\obj\jar/pom.xml'.
1>[INFO] Scanning for projects...
1>[INFO]
1>[INFO] ---------------------< mavenexample:mavenexample >----------------------
1>[INFO] Building mavenexample 1.0.0
1>[INFO] --------------------------------[ jar ]---------------------------------
1>[INFO]
1>[INFO] --- maven-dependency-plugin:2.8:copy-dependencies (default-cli) @ mavenexample ---
1>[INFO] opennlp-brat-annotator-1.9.1.jar already exists in destination.
1>[INFO] aopalliance-repackaged-2.5.0-b30.jar already exists in destination.
1>[INFO] jersey-entity-filtering-2.25.jar already exists in destination.
1>[INFO] opennlp-tools-1.9.1.jar already exists in destination.
1>[INFO] morfologik-stemming-2.1.3.jar already exists in destination.
1>[INFO] javassist-3.20.0-GA.jar already exists in destination.
1>[INFO] jcommander-1.48.jar already exists in destination.
1>[INFO] jersey-guava-2.25.jar already exists in destination.
1>[INFO] jersey-media-json-jackson-2.25.jar already exists in destination.
1>[INFO] jersey-container-grizzly2-http-2.25.jar already exists in destination.
1>[INFO] morfologik-tools-2.1.3.jar already exists in destination.
1>[INFO] hk2-utils-2.5.0-b30.jar already exists in destination.
1>[INFO] jersey-media-jaxb-2.25.jar already exists in destination.
1>[INFO] javax.annotation-api-1.2.jar already exists in destination.
1>[INFO] jackson-module-jaxb-annotations-2.8.4.jar already exists in destination.
1>[INFO] jackson-jaxrs-base-2.8.4.jar already exists in destination.
1>[INFO] opennlp-uima-1.9.1.jar already exists in destination.
1>[INFO] javax.inject-2.5.0-b30.jar already exists in destination.
1>[INFO] morfologik-fsa-2.1.3.jar already exists in destination.
1>[INFO] hk2-api-2.5.0-b30.jar already exists in destination.
1>[INFO] osgi-resource-locator-1.0.1.jar already exists in destination.
1>[INFO] jersey-client-2.25.jar already exists in destination.
1>[INFO] jersey-server-2.25.jar already exists in destination.
1>[INFO] grizzly-http-server-2.3.28.jar already exists in destination.
1>[INFO] javax.ws.rs-api-2.0.1.jar already exists in destination.
1>[INFO] jackson-databind-2.8.4.jar already exists in destination.
1>[INFO] opennlp-morfologik-addon-1.9.1.jar already exists in destination.
1>[INFO] jackson-jaxrs-json-provider-2.8.4.jar already exists in destination.
1>[INFO] hk2-locator-2.5.0-b30.jar already exists in destination.
1>[INFO] hppc-0.7.1.jar already exists in destination.
1>[INFO] jackson-core-2.8.4.jar already exists in destination.
1>[INFO] grizzly-framework-2.3.28.jar already exists in destination.
1>[INFO] validation-api-1.1.0.Final.jar already exists in destination.
1>[INFO] morfologik-fsa-builders-2.1.3.jar already exists in destination.
1>[INFO] jersey-common-2.25.jar already exists in destination.
1>[INFO] grizzly-http-2.3.28.jar already exists in destination.
1>[INFO] jackson-annotations-2.8.4.jar already exists in destination.
1>[INFO] ------------------------------------------------------------------------
1>[INFO] BUILD SUCCESS
1>[INFO] ------------------------------------------------------------------------
1>[INFO] Total time:  1.178 s
1>[INFO] Finished at: 2022-06-09T20:28:03+07:00
1>[INFO] ------------------------------------------------------------------------
1>MavenExample -> F:\Projects\_Demos\MavenReferencePoC\src\MavenExample\bin\Debug\netcoreapp3.1\MavenExample.dll
========== Rebuild All: 1 succeeded, 0 failed, 0 skipped ==========
```