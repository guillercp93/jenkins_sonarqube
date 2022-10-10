pipeline {
   agent any
   parameters{
      string(
         name: "GIT_BRANCH",
         defaultValue: "master",
         description: "Nombre de la rama a tomar del repositorio Git al hacer checkout."
      )
      string(
         name: "SONAR_PROJECT_KEY",
         defaultValue: "base-project-dotnet6",
         description: "Nombre clave del proyecto en SonarQube a reportar el scanner."
      )
   }
   stages {
      stage ('Build API') {
         environment {
            SONAR_SCANNER = tool 'SonarScannerCore'
         }
         steps {
            // Limpiar Workspace principal
            cleanWs()

            ws(dir: '\\var\\jenkins_home\\workspace') {
               // Limpiar Workspace secundario
               cleanWs()

               // Clonar repositorio de la API en la rama deseada
               checkout([  
                  $class: 'GitSCM', 
                  branches: [[name: "refs/heads/${params.GIT_BRANCH}"]], 
                  doGenerateSubmoduleConfigurations: false, 
                  extensions: [[
                     $class: 'RelativeTargetDirectory',
                     relativeTargetDir: 'ProjectsMWS'
                  ]], 
                  submoduleCfg: [], 
                  userRemoteConfigs: [[
                     credentialsId: 'b8c817fc-6d44-4688-be73-624542bea8f9',
                     url: 'https://github.com/guillercp93/DotNetBaseProject.git'
                  ]]
               ])

               // Clonar repositorio de auditoría
               // checkout([  
               //    $class: 'GitSCM', 
               //    branches: [[name: 'refs/heads/develop']], 
               //    doGenerateSubmoduleConfigurations: false, 
               //    extensions: [[
               //       $class: 'RelativeTargetDirectory',
               //       relativeTargetDir: 'PaqueteAuditoria'
               //    ]], 
               //    submoduleCfg: [], 
               //    userRemoteConfigs: [[
               //       credentialsId: 'GITLAB-FANALCA',
               //       url: 'https://gitlab.com/sipresplus/pser-sp-sharedauditrequestresponselibrary-nuget.git'
               //    ]]
               // ])

               script() {
                  def sources = [];

                  def buildDotnet = { String PROJECT ->
                     bat "dotnet clean ${PROJECT} --configuration Release --nologo"
                     bat "dotnet restore ${PROJECT}"
                     bat "dotnet build ${PROJECT} --configuration Release --no-restore --nologo"
                  }

                  def addToSources = { NOMBRE_NUGET_SOURCE, NUGET_FOLDER ->
                     int CONSULTAR_LISTA_FUENTES = bat (
                        script: "dotnet nuget list source",
                        returnStdout: true
                     )
                     .trim()
                     .indexOf("${NOMBRE_NUGET_SOURCE}")

                     // Remover la fuente en caso que exista
                     if (CONSULTAR_LISTA_FUENTES > -1) {
                        bat "dotnet nuget remove source ${NOMBRE_NUGET_SOURCE}" 
                     }

                     bat "dotnet nuget add source ${NUGET_FOLDER} --name \"${NOMBRE_NUGET_SOURCE}\""

                     String addString = "<add key=\"${NOMBRE_NUGET_SOURCE}\" value=\"${NUGET_FOLDER}\" />"

                     sources.add(addString)
                  }

                  // Build PaqueteAuditoria
                  // String PROJECT = "${workspace}\\PaqueteAuditoria\\SharedAuditRequestResponseLibrary\\SharedAuditRequestResponseLibrary.csproj"
                  // String NUGET_FOLDER = "${workspace}\\PaqueteAuditoria\\SharedAuditRequestResponseLibrary\\bin\\Release"
                  // String NOMBRE_NUGET_SOURCE = 'PaqueteAuditoria'

                  // buildDotnet(PROJECT);

                  // addToSources(NOMBRE_NUGET_SOURCE, NUGET_FOLDER)

                  PROJECT = "${workspace}\\API\\DotNetBaseProject\\BaseProject.csproj"

                  // Build ApiLineasVehiculos y generar reporte de SonarQube
                  withSonarQubeEnv('SonarQube') {
                     bat "dotnet ${SONAR_SCANNER}\\SonarScanner.MSBuild.dll begin /k:\"${params.SONAR_PROJECT_KEY}\""
                     buildDotnet(PROJECT)
                     bat "dotnet ${SONAR_SCANNER}\\SonarScanner.MSBuild.dll end"
                  }
               }
            }
         }
      }
   }
}
