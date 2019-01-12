def SERVICE_GROUP = "svc-grp"
def SERVICE_NAME = "svc-name"
def IMAGE_NAME = "${SERVICE_GROUP}-${SERVICE_NAME}"
def REPOSITORY_URL = "https://github.com/gelius7/test-spring-boot.git"
def REPOSITORY_SECRET = ""
def SLACK_TOKEN_DEV = ""
def SLACK_TOKEN_DQA = ""

@Library("github.com/gelius7/valve-butler")
def butler = new com.opsnow.valve.v7.Butler()
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
    stage("Input Parameters") {
      SERVICE_GROUP = input(message:'input service group', parameters: [
            [$class: 'TextParameterDefinition', defaultValue: SERVICE_GROUP, description: 'Service Group', name: 'Service-Group']
        ])
      SERVICE_NAME = input(message:'input service name', parameters: [
            [$class: 'TextParameterDefinition', defaultValue: SERVICE_NAME, description: 'Service Name', name: 'Service-Name']
        ])
      
      REPOSITORY_URL = input(message:'input repository url', parameters: [
            [$class: 'TextParameterDefinition', defaultValue: REPOSITORY_URL, description: 'Repository Url', name: 'Repository-Url']
        ])
      REPOSITORY_SECRET = input(message:'input repository secret', parameters: [
            [$class: 'TextParameterDefinition', defaultValue: REPOSITORY_SECRET, description: 'Repository Secret', name: 'Repository-Secret']
        ])
      SLACK_TOKEN_DEV = input(message:'input slack token for DEV', parameters: [
            [$class: 'TextParameterDefinition', defaultValue: SLACK_TOKEN_DEV, description: 'Slack token for DEV', name: 'Slack-Token-Dev']
        ])
      SLACK_TOKEN_DQA = input(message:'input slack token for QA', parameters: [
            [$class: 'TextParameterDefinition', defaultValue: SLACK_TOKEN_DQA, description: 'Slack token for QA', name: 'Slack-Token-QA']
        ])
        echo ("user input : " + SERVICE_GROUP)
    }
    stage("Prepare") {
      container("builder") {
        butler.prepare(IMAGE_NAME)
      }
    }
    stage("Checkout") {
      container("builder") {
        try {
          if (REPOSITORY_SECRET) {
            git(url: REPOSITORY_URL, branch: BRANCH_NAME, credentialsId: REPOSITORY_SECRET)
          } else {
            git(url: REPOSITORY_URL, branch: BRANCH_NAME)
          }
        } catch (e) {
          butler.failure(SLACK_TOKEN_DEV, "Checkout")
          throw e
        }
      }
    }
    stage("Deploy OKC1") {
      container("builder") {
        try {
          // deploy(cluster, namespace, sub_domain, profile)
          butler.deploy_prd("okc1", "${SERVICE_GROUP}-prod", "${IMAGE_NAME}-stage", "prd")
          butler.success([SLACK_TOKEN_DEV,SLACK_TOKEN_DQA], "Deploy OKC1")
        } catch (e) {
          butler.failure([SLACK_TOKEN_DEV,SLACK_TOKEN_DQA], "Deploy OKC1")
          throw e
        }
      }
    } 
  }
}
