@Library(['nbs-homehub-jenkins-shared-library','api-ci-common-shared-library@homeapi_cloudmigration'])

def HELM_REPO = 'https://artifactory.aws.nbscloud.co.uk:443/artifactory/api/helm/'
def HELM_URL = 'https://artifactory.aws.nbscloud.co.uk/artifactory/'
def MANIFEST_REPO_NAME ='homehub-docker-dev'
def HELM_REPO_NAME = 'homehub-helm-dev'
def ARTIFACTORY_USER_KEY = 'artifactory_homehub_ci_user'
def ARTIFACTORY_TOKEN_KEY= 'artifactory_homehub_ci_token'
def IMAGE_BRANCH = ""
def IMAGE_COMMIT = ""
def DEPLOYMENT_LIST = []
def SOURCE_CLUSTER_NAME = "${clusterSelection(params.SOURCE_ENVIRONMENT, params.SOURCE_CLUSTER)}"
def DESTINATION_CLUSTER_NAME = "${clusterSelection(params.DESTINATION_ENVIRONMENT, params.DESTINATION_CLUSTER)}"

pipeline {
    
    options {
        skipDefaultCheckout()
    }

    parameters {
        choice(
            name: 'SOURCE_ENVIRONMENT',
            choices: ['DEV', 'TEST'],
            description: 'Environment to get the service details'
        )
        choice(
            name: 'SOURCE_CLUSTER',
            choices: ['default', 'alternate'],
            description: "Cluster to get the service details"
        )
        choice(
            name: 'DESTINATION_ENVIRONMENT',
            choices: ['DEV', 'TEST'],
            description: 'Environment deploy the services'
        )
        choice(
            name: 'DESTINATION_CLUSTER',
            choices: ['default', 'alternate'],
            description: 'Cluster to deploy the services')
        choice(
            name: 'SOURCE_NAMESPACE',
            choices: [
                'api-dev-01',
                'api-dev-02',
                'api-dev-03',
                'api-dev-04',
                'api-dev-05',
                'api-dev-06',
                'api-dev-07',
                'api-dev-08',
                'api-sit-01',
                'api-sit-02',
                'api-sit-03',
                'api-sit-04',
                'api-sit-05',
                'api-pre-01',
                'api-pre-02',
                'api-pre-03',
                'dip-perf-01',
                'dip-perf-02',
                'dip-perf-03'
            ],
            description: 'Namespace to get the service details'
        )
        choice(
            name: 'DESTINATION_NAMESPACE',
            choices: [      
                'api-dev-01',
                'api-dev-02',
                'api-dev-03',
                'api-dev-04',
                'api-dev-05',
                'api-dev-06',
                'api-dev-07',
                'api-dev-08',
                'api-sit-01',
                'api-sit-02',
                'api-sit-03',
                'api-sit-04',
                'api-sit-05',
                'api-pre-01',
                'api-pre-02',
                'api-pre-03',
                'dip-perf-01',
                'dip-perf-02',
                'dip-perf-03'
            ],
            description: 'Namespace to deploy the services'
        )
        booleanParam(defaultValue: true, name: 'Mortgages_CopyTextManager', description: 'repo:nbs-mortgages-copytextmanager')
        booleanParam(defaultValue: true, name: 'Home_Customer_Manager', description: 'repo:home-customermanager')
        booleanParam(defaultValue: true, name: 'Mortgages_Application', description: 'repo:nbs-mortgages-application')
        booleanParam(defaultValue: true, name: 'DIP_Service', description: 'repo:nbs-mortgages-dip')
        booleanParam(defaultValue: true, name: 'DIP_Certificate_Service', description: 'repo:nbs-mortgages-dip-certificate')
        booleanParam(defaultValue: true, name: 'Healthcheck_Webapp', description: 'repo:nbs-mortgages-healthcheck-webapp')
        booleanParam(defaultValue: true, name: 'Intermediary_AuthManager', description: 'repo:nbs-mortgages-intermediary-authmanager')
        booleanParam(defaultValue: true, name: 'Intermediary_CreateCase', description: 'repo:nbs-mortgages-intermediary-createcase')
        booleanParam(defaultValue: true, name: 'Intermediary_CreateCase_v14', description: 'repo:nbs-mortgages-intermediary-createcase-v14')
        booleanParam(defaultValue: true, name: 'Remo_AffordabilityManager', description: 'repo:nbs-mortgages-remo-affordabilitymanager')
        booleanParam(defaultValue: true, name: 'DIP_PropertyValuer', description: 'repo:nbs-mortgages-dip-propertyvaluer')
        booleanParam(defaultValue: true, name: 'MsoCreateCase', description: 'repo:nbs-mortgages-msocreatecase')
        booleanParam(defaultValue: true, name: 'Remo_Crud_Api', description: 'repo:nbs-mortgages-remo-crud-api')
        booleanParam(defaultValue: true, name: 'Sandbox_Stub', description: 'repo:nbs-mortgages-sandbox-stub')
        booleanParam(defaultValue: true, name: 'Stub_GetMortgageDetails', description: 'repo:nbs-mortgages-stub-getmortgagedetails')
        booleanParam(defaultValue: true, name: 'Stub_Icm', description: 'repo:nbs-mortgages-stub-icm')
        booleanParam(defaultValue: true, name: 'Stub_PrismPropertyValuator', description: 'repo:nbs-mortgages-stub-prismpropertyvaluator')
    }

    agent {
        kubernetes {
            cloud "kubernetes-${SOURCE_CLUSTER_NAME}"
            defaultContainer 'helm3'
            yaml helm3Pod()
        }
    }

    stages {
        stage('Get Deployments') {
            steps {
                script {
                    getSelectedServices().each { service ->
                        DEPLOYMENT_LIST.add(getDeploymentDetails(service, params.SOURCE_NAMESPACE))
                    }

                    DEPLOYMENT_LIST.each { service ->
                        println "${service.serviceName},${service.namespace},${service.imageTag},${service.imageCommit},${service.imageBranch}"
                    }
                }
            }
        }

        stage('Deploy') {
            agent {
                kubernetes {
                    cloud "kubernetes-${DESTINATION_CLUSTER_NAME}"
                    defaultContainer 'helm3'
                    yaml helm3Pod()
                }
            }
            steps {
                script {
                    withVault(vaultSecrets: [[engineVersion: 2, path: 'secrets/cbjenkins', secretValues: [
                          [envVar: 'ARTIFACTORY_USER', vaultKey: ARTIFACTORY_USER_KEY],
                          [envVar: 'ARTIFACTORY_TOKEN', vaultKey: ARTIFACTORY_TOKEN_KEY]]]])
                    {
                        container(name: 'jnlp') {
                            if (DESTINATION_CLUSTER_NAME.contains("test")) {
                                MANIFEST_REPO_NAME ='homehub-docker-stg'
                                HELM_REPO_NAME = 'homehub-helm-stg'
                            }

                            DEPLOYMENT_LIST.each { service ->
                                sh """
                                    curl -sSf -H 'Authorization: Bearer ${ARTIFACTORY_TOKEN}' -O '${HELM_URL}${HELM_REPO_NAME}/${service.serviceName}/${service.imageTag}/${service.serviceName}-${service.imageTag}.tgz'
                                    tar -xf ${service.serviceName}-${service.imageTag}.tgz
                                    rm -rf ${service.serviceName}-${service.imageTag}.tgz
                                """
                            }
                            sh """
                                ls -l
                            """
                        }
                        container(name: 'helm3') {
                            sh """
                                # TODO: Helm requires write access to directorys in home. Temp solution is to set dirs to temp.
                                export HELM_CONFIG_HOME=/tmp
                                export HELM_CACHE_HOME=/tmp
                                export HELM_DATA_HOME=/tmp
                                
                                helm repo add ${HELM_REPO_NAME} ${HELM_REPO}${HELM_REPO_NAME} \
                                    --username ${ARTIFACTORY_USER} \
                                    --password ${ARTIFACTORY_TOKEN} \
                                    --insecure-skip-tls-verify \
                                    --debug
                                
                                helm repo update
                            """
                            DEPLOYMENT_LIST.each { service ->
                                sh """
                                    helm upgrade ${service.serviceName} ${HELM_REPO_NAME}/${service.serviceName} \
                                    --values ${service.serviceName}/values/${DESTINATION_NAMESPACE}.yaml \
                                    --namespace=${DESTINATION_NAMESPACE} \
                                    --username ${ARTIFACTORY_USER} \
                                    --password ${ARTIFACTORY_TOKEN} \
                                    --insecure-skip-tls-verify  \
                                    --version ${service.imageTag} \
                                    --debug --install --force \
                                    --set IMAGE_TAG=${service.imageTag} \
                                    --set IMAGE_BRANCH=${service.imageBranch} \
                                    --set IMAGE_COMMIT=${service.imageCommit} \
                                    --set CLUSTER=${DESTINATION_CLUSTER_NAME}
                                """
                            }
                        }
                    }
                }
            }
        }
    }
}

def getSelectedServices() {
    def selectedServices = []

    if (params.Mortgages_CopyTextManager) {
        selectedServices.add('nbs-mortgages-copytextmanager')
    }
    if (params.Home_Customer_Manager) {
        selectedServices.add('home-customermanager')
    }
    if (params.Mortgages_Application) {
        selectedServices.add('nbs-mortgages-application')
    }
    if (params.DIP_Service) {
        selectedServices.add('nbs-mortgages-dip')
    }
    if (params.DIP_Certificate_Service) {
        selectedServices.add('nbs-mortgages-dip-certificate')
    }
    if (params.Remo_AffordabilityManager) {
        selectedServices.add('nbs-mortgages-remo-affordabilitymanager')
    }
    if (params.DIP_PropertyValuer) {
        selectedServices.add('nbs-mortgages-dip-propertyvaluer')
    }
    if (params.Healthcheck_Webapp) {
        selectedServices.add('nbs-mortgages-healthcheck-webapp')
    }
    if (params.Intermediary_AuthManager) {
        selectedServices.add('nbs-mortgages-intermediary-authmanager')
    }
    if (params.Intermediary_CreateCase) {
        selectedServices.add('nbs-mortgages-intermediary-createcase')
    }
    if (params.Intermediary_CreateCase_v14) {
        selectedServices.add('nbs-mortgages-intermediary-createcase-v14')
    }
    if (params.MsoCreateCase) {
        selectedServices.add('nbs-mortgages-msocreatecase')
    }
    if (params.Remo_Crud_Api) {
        selectedServices.add('nbs-mortgages-remo-crud-api')
    }
    if (params.Sandbox_Stub) {
        selectedServices.add('nbs-mortgages-sandbox-stub')
    }
    if (params.Stub_GetMortgageDetails) {
        selectedServices.add('nbs-mortgages-stub-getmortgagedetails')
    }
    if (params.Stub_Icm) {
        selectedServices.add('nbs-mortgages-stub-icm')
    }
    if (params.Stub_PrismPropertyValuator) {
        selectedServices.add('nbs-mortgages-stub-prismpropertyvaluator')
    }
    
    return selectedServices
}

def getDeploymentDetails(service, namespace) {
    def deploymentYaml = sh (returnStdout: true, label: 'Get deployment details', script: "helm get values $service -a -n $namespace -oyaml")
    def deploymentDetails = readYaml text: deploymentYaml

    def deploymentMap = [
        serviceName: deploymentDetails.SERVICE_NAME,
        namespace: deploymentDetails.NAMESPACE,
        imageTag: deploymentDetails.IMAGE_TAG,
        imageCommit: deploymentDetails.IMAGE_COMMIT,
        imageBranch: deploymentDetails.IMAGE_BRANCH
    ]

    return deploymentMap
}
