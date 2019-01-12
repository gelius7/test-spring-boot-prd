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
            [$class: 'TextParameterDefinition', defaultValue: '${SERVICE_GROUP}', description: 'Service Group', name: 'Service-Group']
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
