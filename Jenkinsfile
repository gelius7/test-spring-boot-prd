def SERVICE_GROUP = "svc-grp"
def SERVICE_NAME = "svc-name"
def IMAGE_NAME = "${SERVICE_GROUP}-${SERVICE_NAME}"
def REPOSITORY_URL = "https://github.com/gelius7/test-spring-boot.git"
def REPOSITORY_SECRET = ""
def SLACK_TOKEN_DEV = ""
def SLACK_TOKEN_DQA = ""

@Library("github.com/gelius7/valve-butler")
def butler = new com.opsnow.valve.v8.Butler()
def label = "worker-${UUID.randomUUID().toString()}"

properties([
  buildDiscarder(logRotator(daysToKeepStr: "60", numToKeepStr: "30"))
])
podTemplate(label: label, containers: [
  containerTemplate(name: "builder", image: "quay.io/opsnow-tools/valve-builder", command: "cat", ttyEnabled: true, alwaysPullImage: true),
  containerTemplate(name: "maven", image: "maven:3.5.4-jdk-8-alpine", command: "cat", ttyEnabled: true)
], volumes: [
  // hostPathVolume(mountPath: "/var/run/docker.sock", hostPath: "/var/run/docker.sock"),
  // hostPathVolume(mountPath: "/home/jenkins/.draft", hostPath: "/home/jenkins/.draft"),
  // hostPathVolume(mountPath: "/home/jenkins/.helm", hostPath: "/home/jenkins/.helm")
]) {
  node(label) {
    stage("Prepare") {
      container("builder") {
        def userInput = input(message:'test', parameters: [
//              [$class: 'TextParameterDefinition', defaultValue: 'default', description: 'Describe', name: 'defname']
            [$class: 'ChoiceParameterDefinition', choices: "default\n222ddd\nddd333", description: 'test select one', name: 'firstParam']
        ])
        echo ("user input : " + userInput)
      }
    } 
    stage("Checkout") {
      container("builder") {
        def userInput = input(message:'test', parameters: [
              [$class: 'TextParameterDefinition', defaultValue: 'default', description: 'Describe', name: 'defname']
        ])
        echo ("user input : " + userInput)
      }
    } 
  }
}
