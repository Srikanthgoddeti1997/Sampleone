@Library('nbs-homehub-jenkins-shared-library')

def HELM_REPO = 'https://artifactory.aws.nbscloud.co.uk:443/artifactory/api/helm/'
def HELM_URL = 'https://artifactory.aws.nbscloud.co.uk/artifactory/'
def MANIFEST_REPO_NAME ='homehub-docker-dev'
def HELM_REPO_NAME = 'homehub-helm-dev'
def IMAGE_BRANCH = ""
def IMAGE_COMMIT = ""
def DEPLOYMENT_LIST = []
def CLUSTER_NAME = "${clusterSelection(params.ENVIRONMENT, params.CLUSTER)}"
def jobApprovalDuration = 10
def jobApprovalTimeUnit = "MINUTES"
def DEV_NAMESPACES = [
                "api-dev-01",
                "api-dev-02",
                "api-dev-03",
                "api-dev-04",
                "api-dev-05",
                "api-dev-06",
                "api-dev-07",
                "api-dev-08",
                "api-sit-01",
                "api-sit-02",
                "api-sit-03",
                "api-sit-04",
                "api-sit-05",
                "dip-dev-01",
                "dip-dev-02",
                "dip-dev-03",
                "dip-dev-04",
                "dip-dev-05",
                "dip-dev-06",
                "dip-dev-07",
                "dip-dev-08",
                "dip-sit-01",
                "dip-sit-02",
                "dip-sit-03",
                "dip-sit-04",
                "dip-sit-05",
                "dip-dev-common-01",
                "dip-dev-common-02",
                "dip-dev-common-03",
                "dip-dev-common-04",
                "dip-dev-common-05",
                "dip-dev-common-06",
                "dip-dev-common-07",
                "dip-dev-common-08",
                "dip-sit-common-01",
                "dip-sit-common-02",
                "dip-sit-common-03",
                "dip-sit-common-04",
                "dip-sit-common-05"   
            ]
def TEST_NAMESPACES = [
                "api-pre-01",
                "api-pre-02",
                "api-pre-03",
                "api-perf-01",
                "api-perf-02",
                "api-perf-03",
                "dip-pre-01",
                "dip-pre-02",
                "dip-pre-03",
                "dip-perf-01",
                "dip-perf-02",
                "dip-perf-03",
                "dip-pre-common-01",
                "dip-pre-common-02",
                "dip-pre-common-03",
                "dip-perf-common-01",
                "dip-perf-common-02",
                "dip-perf-common-03"
            ]

pipeline {
    
    options {
        skipDefaultCheckout()
    }

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['DEV', 'TEST'],
            description: 'Environment to run pipeline'
        )
        choice(
            name: 'CLUSTER',
            choices: ['default','alternate'],
            description: 'Cluster name to get the service details.'
        )
    }

    agent {
        kubernetes {
            cloud "kubernetes-${CLUSTER_NAME}"
            defaultContainer 'helm3'
            yaml helm3Pod()
        }
    }

    stages {
        stage('Get Deployments') {
            steps {
                script {
                    if (CLUSTER_NAME.contains("test")) {
                        TEST_NAMESPACES.each { namespace ->
                            sh(label: "current deployments in ${namespace} on ${CLUSTER_NAME}", script: "helm list -n ${namespace}")
                        }
                    }
                    else {
                        DEV_NAMESPACES.each { namespace ->
                            sh(label: "current deployments in ${namespace} on ${CLUSTER_NAME}", script: "helm list -n ${namespace}")
                        }          
                    }
                }
            }
        }
        stage('Non-production Approval') {
            steps {
                script {
                    getJobApproval(jobApprovalDuration, jobApprovalTimeUnit)
                    sh "echo 'Uninstalling deployments on ${CLUSTER_NAME}'"
                }
            }
        }
        stage('Uninstall helm charts') {
            steps {
                script {
                    if (CLUSTER_NAME.contains("test")) {
                        TEST_NAMESPACES.each { namespace ->
                            def allDeployments = sh (returnStdout: true, label: "Get deployment details", script: "helm list -n ${namespace} --short")
                            println(allDeployments)
                            if (allDeployments.size() > 0){
                                sh(label: "deleting all helm releases in ${namespace}", script: "helm uninstall -n ${namespace} \$(helm list -n ${namespace} --short)")
                            }   
                        }
                    }
                    else {
                        DEV_NAMESPACES.each { namespace ->
                            def allDeployments = sh (returnStdout: true, label: "Get deployment details", script: "helm list -n ${namespace} --short")
                            print(allDeployments)
                            if (allDeployments.size() > 0){
                                sh(label: "deleting all helm releases in ${namespace}", script: "helm uninstall -n ${namespace} \$(helm list -n ${namespace} --short)")
                            }
                        }          
                    }
                }
            }
        }
    }
}

def getJobApproval(int jobApprovalDuration, String jobApprovalTimeUnit) {
    echo "Inside get job approval method"

    try {
        def approvalMessage = "This will un-install all deployments on the cluster for selected namespaces. Do you want to proceed?"
        this.timeout(time: jobApprovalDuration, unit: jobApprovalTimeUnit) {
            this.input(id: 'Approval', message: approvalMessage, submitter: deployApprovers.list())
        }
    } catch (err) {
        this.error("The build approval request has failed. Error details: ${err}")
    }
}