NAMESPACE="cap-apps"
NOMBRE_APP="my-app"
GIT_REPO=""
TARGET_PORT="8080"
GIT_USERNAME=""
GIT_PASSWORD=""
APPROVAL_EMAIL=""
GIT_REF=""

def templateName = 'eap71-basic-s2i'
def projectDev = "${NAMESPACE}"
def projectQA = "${NAMESPACE}-qa"
def projectProd = "${NAMESPACE}-production"
def ocpClusterName = "master"
def bc = ""
def dc = ""
try {
    timeout(20) {
		stage('preamble') {
			openshift.withCluster(ocpClusterName) {
				openshift.withProject(projectDev) {
					echo "[PREAMBLE]Using project: ${openshift.project()}"
				}
			}
		}
                stage('cleanup') {
                        openshift.withCluster(ocpClusterName) {
                                openshift.withProject(projectDev) {
                                    echo "[CleanUP]Using project: ${openshift.project()}"
                                    // delete everything with this template label
                                    openshift.selector("all", [template:${templateName}]).delete()
                                    // delete any secrets with this template label
                                    if (openshift.selector("secrets", templateName).exists()) {
                                        openshift.selector("secrets", templateName).delete()
                                    }
                                }
                        }
                }
		stage('create') {
			openshift.withCluster(ocpClusterName) {
				//openshift.verbose()
				openshift.withProject(projectDev) {
					echo '[CREATE]Ejecutando create'
					def res
						echo '[CREATE]Proyecto no existe, lo creamos a partir del template'
						def params = "APPLICATION_NAME=${NOMBRE_APP}"
						echo "[CREATE] PARAMS ${params}"
						res = openshift.newApp(templateName, params)

						res = openshift.apply(openshift.process( template, params ) );
					bc = res.narrow('bc')
					dc = res.narrow('dc')
				}
				//openshift.verbose(false)
			}
		}
		stage('build') {
			openshift.withCluster(ocpClusterName) {
				//openshift.verbose()
				openshift.withProject(projectDev) {
					echo "[BUILD]Ejecutando build ${bc}"
					def currentBuild = bc.startBuild()
					def builds = openshift.selector("build ${currentBuild.object().metadata.name}")
					//echo "[BUILD] BUILDS : ${builds}"
					timeout(5) {
						builds.untilEach(1) {
							echo "PHASE BUILD: ${it.object().metadata.name}"
							return (it.object().status.phase == "Complete")
						}
					}
				}
				//openshift.verbose(false)
			}
		}
		stage('deploy') {
			openshift.withCluster(ocpClusterName) {
				//openshift.verbose()
				openshift.withProject(projectDev) {
					echo '[DEPLOY]Ejecutando deploy'
				    def rm = openshift.selector("dc/${NOMBRE_APP}").rollout().latest()
				    timeout(5) {
					  openshift.selector("dc/${NOMBRE_APP}").related('pods').untilEach(1) {
					    return (it.object().status.phase == "Running")
					  }
				    }
				}
				//openshift.verbose(false)
			}
		}
	}
} catch (Exception e ) {
    echo "Error: ${e}"
}
