#!groovy

node {
   // ------------------------------------
   // -- ETAPA: Compilar
   // ------------------------------------
   stage 'Compilar'
   
   // -- Configura variables
   echo 'Configurando variables'
   def mvnHome = tool 'Maven3'
   env.PATH = "${mvnHome}/bin:${env.PATH}"
   echo "var mvnHome='${mvnHome}'"
   echo "var env.PATH='${env.PATH}'"
   
   // -- Descarga código desde SCM
   echo 'Descargando código de SCM'
   sh 'rm -rf *'
   checkout scm
   
   // -- Compilando
   echo 'Compilando aplicación'
   sh 'mvn clean compile'
   
   // ------------------------------------
   // -- ETAPA: Test
   // ------------------------------------
   stage 'Test'
   echo 'Ejecutando tests'
   try{
      sh 'mvn verify'
      step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
   }catch(err) {
      step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
      if (currentBuild.result == 'UNSTABLE')
         currentBuild.result = 'FAILURE'
      throw err
   }
   
   // ------------------------------------
   // -- ETAPA: Instalar
   // ------------------------------------
   stage 'Instalar'
   echo 'Instala el paquete generado en el repositorio maven'
   sh 'mvn install -Dmaven.test.skip=true'
   
   // ------------------------------------
   // -- ETAPA: Archivar
   // ------------------------------------
   stage 'Archivar'
   echo 'Archiva el paquete el paquete generado en Jenkins'
   step([$class: 'ArtifactArchiver', artifacts: '**/target/*.jar, **/target/*.war', fingerprint: true])
   
   // ------------------------------------
   // -- ETAPA: Cobertura de código
   // ------------------------------------
   stage 'Code Coverage (Cobertura de codigo)' 
   echo 'Comprueba la cobertura que hacen los test sobre el código desarrollado'
   step([$class: 'JacocoPublisher', execPattern: '**/**.exec', exclusionPattern: '**/*Test*.class'])
   
   checkout scm
   def mavenSettingsFile = " ${mvnHome}/conf/settings.xml"
	sh "mvn -s ${mavenSettingsFile} clean source:jar javadoc:javadoc checkstyle:checkstyle pmd:pmd findbugs:findbugs package"
	step([$class: 'WarningsPublisher', consoleParsers: [[parserName: 'Maven']]])
	step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
   // ------------------------------------
   // -- ETAPA: CheckStyle
   // ------------------------------------
   	stage 'CheckStyle'
   	step([$class: 'hudson.plugins.checkstyle.CheckStylePublisher', pattern: '**/target/checkstyle-result.xml'])
   
   // ------------------------------------
   // -- ETAPA: PMD
   // ------------------------------------
   	stage 'PmdPublisher'
   	step([$class: 'PmdPublisher'])
   
   // ------------------------------------
   // -- ETAPA: AnalysisPublisher
   // ------------------------------------
   	stage 'AnalysisPublisher'
	step([$class: 'AnalysisPublisher'])
   

}
