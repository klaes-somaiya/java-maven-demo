# Practical 2: Jenkins Freestyle job with Maven

**Aim:** Install Jenkins, then create a Freestyle job `java-build-job` that pulls a Java Maven project from GitHub, checks GitHub for new commits every 2 minutes (Poll SCM), and builds the project with Maven.

**What this practical does:** **Jenkins** is an automation server for **CI (Continuous Integration)**: every time code changes, Jenkins builds and tests it automatically. A **Freestyle job** is set up through forms. **Maven** reads `pom.xml`, downloads libraries (JUnit), compiles, tests and packages the code into a `.jar`.

**Where to do this:**

- **Part A:** PowerShell as Administrator, the downloaded installers, then Chrome at `http://localhost:8080`.
- **Part B:** Chrome, on github.com.
- **Parts C to E:** Chrome, at `http://localhost:8080`.

## Part A: Install and start Jenkins (Q2.1, 6 marks)

1.  **Install Java 21.** Open PowerShell **as Administrator** and run:

    ```powershell
    winget install EclipseAdoptium.Temurin.21.JDK
    ```

    Close PowerShell, open a new one and run `java -version`. It should print a version starting with 21. If `winget` isn’t recognized, download **Temurin 21 (.msi)** from adoptium.net and tick **Set JAVA_HOME** during the install.
2.  **Install Git.** Download **Git for Windows** from git-scm.com and install it with the defaults. Check it with `git --version`.
3.  **Download Jenkins.** Go to **jenkins.io/download** and, under **LTS**, download the **Windows** installer (.msi).
4.  **Run the installer.**
    - **Logon type:** Run service as LocalSystem
    - **Port:** 8080. Click **Test Port**, then **Next**.
    - **Java home:** choose the JDK 21 folder, for example `C:\Program Files\Eclipse Adoptium\jdk-21...`
    - Click **Install**, then **Finish**.
5.  **Unlock Jenkins.** Open **http://localhost:8080**. Open this file in Notepad, copy the password inside it and paste it into the page: `C:\ProgramData\Jenkins\.jenkins\secrets\initialAdminPassword`. Click **Continue**.
6.  **Install suggested plugins.** Click **Install suggested plugins** and wait until every plugin shows a green tick.
7.  **Create the admin user.** Fill in the username, password, full name and email, then click **Save and Continue**. Write the password down.
8.  **Finish setup.** Keep the URL as `http://localhost:8080/`, click **Save and Finish**, then **Start using Jenkins**.
9.  **Restart Jenkins** so it can find Git. In PowerShell as Administrator, run `net stop jenkins`, then `net start jenkins`.

**Screenshots:** `java -version`, the Unlock Jenkins page, plugins installing, Create First Admin User, and the Jenkins dashboard.

## Part B: The Java Maven project on GitHub

The project is ready in this repo, with a README containing every step:

```text
https://github.com/show-star/aa2.git
```

To make your own copy instead, create a new **Public** repo on github.com, then use **Add file → Create new file** for each file below. Typing `/` in the file name creates folders.

**pom.xml**

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>java-maven-demo</artifactId>
  <version>1.0</version>
  <packaging>jar</packaging>

  <properties>
    <maven.compiler.release>17</maven.compiler.release>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.13.2</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

**src/main/java/com/example/App.java**

```java
package com.example;

public class App {
    public static int add(int a, int b) {
        return a + b;
    }

    public static void main(String[] args) {
        System.out.println("Hello from Jenkins Maven build! 2 + 3 = " + add(2, 3));
    }
}
```

**src/test/java/com/example/AppTest.java**

```java
package com.example;

import static org.junit.Assert.assertEquals;
import org.junit.Test;

public class AppTest {
    @Test
    public void testAdd() {
        assertEquals(5, App.add(2, 3));
    }
}
```

## Part C: Add Maven to Jenkins (one time, important)

1.  In Jenkins, click **Manage Jenkins**, then **Tools**.
2.  Scroll down to **Maven installations** and click **Add Maven**.
3.  **Name:** `Maven3` (type exactly this).
4.  Tick **Install automatically** and keep the latest **3.9.x** version.
5.  Click **Save**.

You don’t need to install Maven on Windows: Jenkins downloads it on the first build.

## Part D: Create the Freestyle job (Q2.2 to Q2.4)

1.  Click **New Item**, name it `java-build-job`, select **Freestyle project** and click **OK**.
2.  **Source Code Management:** select **Git**.
    - **Repository URL:** `https://github.com/show-star/aa2.git` (or your own repo)
    - **Branch Specifier:** change `*/master` to `*/main`
3.  **Triggers:** tick **Poll SCM**. **Schedule:** `H/2 * * * *` (check GitHub every 2 minutes).
4.  **Build Steps:** click **Add build step**, then **Invoke top-level Maven targets**.
    - **Maven Version:** **Maven3**. Don’t leave it on (Default).
    - **Goals:** `clean package`
5.  Click **Save**.

**Screenshot:** the job configuration (Git URL and branch, Poll SCM, Maven goals).

## Part E: Run the build and check Console Output (Q2.5, 7 marks)

1.  Click **Build Now**. Under Build History, click **#1**, then **Console Output**.
2.  The first build takes a few minutes, because Jenkins downloads Maven and JUnit. It should end with:

    ```text
    [INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
    [INFO] Building jar: ...\workspace\java-build-job\target\java-maven-demo-1.0.jar
    [INFO] BUILD SUCCESS
    Finished: SUCCESS
    ```
3.  **Show Poll SCM working:** on GitHub, edit `App.java` (change the message) and commit. Within 2 minutes, build **#2** starts by itself, and its page says **Started by an SCM change**.
4.  **Show fixing a failed build (optional):** change `assertEquals(5, ...)` to `6` in `AppTest.java` and commit. The build turns red with `Failures: 1`. Change it back to `5`, commit, and the next build is green again.

**Screenshots:** Console Output with BUILD SUCCESS, and build \#2 showing *Started by an SCM change*.

## Errors and fixes

| Error in Console Output                                      | Fix                                                                                                                    |
|--------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------|
| `'mvn' is not recognized as an internal or external command` | **Maven Version** was left on (Default). Do Part C, then **Configure** the job and set **Maven Version** to **Maven3** |
| `couldn't find remote ref refs/heads/master`                 | Set **Branch Specifier** to `*/main`                                                                                   |
| `Cannot run program "git"`                                   | Run `net stop jenkins`, then `net start jenkins` (as Administrator)                                                    |
| `release version 17 not supported`                           | Jenkins is running on an old Java. Install Java 21 and point Jenkins at it                                             |
| `there is no POM in this directory`                          | `pom.xml` must be at the top of the repo                                                                               |
| `Could not transfer artifact ... repo.maven.apache.org`      | The network blocks Maven downloads. Switch to a phone hotspot and build again                                          |
| Build \#2 never starts on its own                            | Check that **Poll SCM** is ticked with `H/2 * * * *`, and that you committed a change                                  |

## Viva points

| Question                      | Answer                                                                                               |
|-------------------------------|------------------------------------------------------------------------------------------------------|
| What is Jenkins?              | An open-source automation server for CI/CD                                                           |
| What is CI?                   | Continuous Integration: every change is built and tested automatically                               |
| What is a Freestyle job?      | A Jenkins job set up through forms rather than a script                                              |
| What is Maven?                | A Java build tool that manages libraries through `pom.xml` and compiles, tests and packages the code |
| What does `clean package` do? | Deletes old output, then compiles, tests and creates the `.jar`                                      |
| What is Poll SCM?             | Jenkins checks Git on a schedule and builds when there’s a new commit                                |
| What does `H/2 * * * *` mean? | Every 2 minutes. `H` spreads the start times of different jobs                                       |

**Conclusion:** We installed Jenkins, created a Freestyle job connected to a Maven project on GitHub, set Poll SCM to every 2 minutes, and built it with `mvn clean package`. The build passed its test, and a commit on GitHub started a new build automatically. This shows Continuous Integration with Jenkins.
