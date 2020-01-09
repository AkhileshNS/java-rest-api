# JAVA REST API

- **Step 1**: To install java and gradle, run

  - windows - `choco install ojdkbuild gradle -y`
  - macos - `brew install homebrew/cask/java gradle`
  - linux -

    ```bash
    # Execute commands one after another
    sudo apt install openjdk-8-jdk

    wget https://services.gradle.org/distributions/gradle-5.1-bin.zip -P /tmp
    sudo unzip -d /opt/gradle /tmp/gradle-5.1-bin.zip

    # This command will open the gradle.sh file
    # inside the file paste:
    # export GRADLE_HOME=/opt/gradle/gradle-5.1
    # export PATH=${GRADLE_HOME}/bin:${PATH}
    sudo nano /etc/profile.d/gradle.sh

    sudo chmod +x /etc/profile.d/gradle.sh
    source /etc/profile.d/gradle.sh
    ```

  [OR] download and install openjdk from [here](https://jdk.java.net/) and download and install gradle by following instructions [here](https://gradle.org/install/#manually)

  Verify everything is installed using `java --version` and `gradle --version`

- **Step 2**: Create a new folder and open a bash/cmd/powershell window in there and run `gradle init --type=java-application`

- **Step 3**: The previous step will create a java project for you, navigate to _gradle.build_ in the same folder and in dependencies add:-

  ```gradle
  ...

  dependencies {
    implementation "org.slf4j:slf4j-simple:1.6.1" // ADD THIS
    implementation "com.sparkjava:spark-core:2.8.0" // ADD THIS

    ...
  }
  ```

- **Step 4**: Now navigate to _src/main/java/packagename/App.java_ and add in the contents:-

  ```java
  package packagename;

  /*Add These*/
  import static spark.Spark.*;

  import spark.Request;
  import spark.Response;
  import spark.Route;
  /**/

  public class App {
      public static void main(String[] args) {
          /*Add These*/
          get("/hello", new Route() {
              @Override
              public Object handle(Request request, Response response) throws Exception {
                  return "world of java";
              }
          });
          /**/
      }
  }
  ```

- **Step 5**: Finally run `./gradlew run` to launch your REST API

- **Step 6**: Don't forget use git to manage your project

# To Deploy REST API via Heroku

- **Step 1**: Before deploying to Heroku, since Heroku uses its own port, we need to change the code in App.java to:

  ```java
  package restapi;

  import static spark.Spark.*;

  import spark.Request;
  import spark.Response;
  import spark.Route;

  public class App {
      public static void main(String[] args) {
          /*ADD THIS*/
          ProcessBuilder process = new ProcessBuilder();
          Integer PORT;
          if (process.environment().get("PORT") != null) {
              PORT = Integer.parseInt(process.environment().get("PORT"));
          } else {
              PORT = 5000;
          }
          port(PORT);
          /**/

          get("/hello", new Route() {
              @Override
              public Object handle(Request request, Response response) throws Exception {
                  return "world of java";
              }
          });
      }
  }
  ```

- **Step 2**: Also add the following to your build.gradle:-

  ```gradle
  // Just after plugins
  applicationName = "restapi"

  // At the end
  defaultTasks = ['clean']
  task stage(dependsOn: ['clean', 'installApp'])
  ```

- **Step 2**: Now add a _Procfile_ in your project with the contents:-

  ```Procfile
  web: ./build/install/restapi/bin/restapi
  ```

- **Step 3**: Now create a folder _.github/workflows_ and in there create a file _main.yml_ with contents:

  ```yaml
  name: Deploy
  on:
    push:
      branches:
        - master
  jobs:
    build:
      runs-on: ubuntu-latest

      steps:
        - uses: actions/checkout@v1.0.0
        - uses: akhileshns/heroku-deploy@master
          with:
            heroku_api_key: ${{secrets.HEROKU_API_KEY}}
            heroku_email: ${{secrets.HEROKU_EMAIL}}
            heroku_app_name: ${{secrets.HEROKU_APP_NAME}}
  ```

- **Step 4**: Now we can push this to GitHub but before that, make sure you have created a Heroku account and in account settings, copy the api key. Then in the github repo for this project, go to settings and add secrets HEROKU_API_KEY (Your copied apikey), HEROKU EMAIL (The email associated with your heroku account) and HEROKU_APP_NAME (The name of your app and keep in mind it needs to be unique in heroku)

  Now whenever you push to the master branch of your github repo, your app is automatically deployed to heroku
