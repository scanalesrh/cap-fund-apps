NAMESPACE="myproject"
NOMBRE_APP="eap-app"
APPROVAL_EMAIL=""

def templateName = "eap71-basic-s2i"
def templatePath = "eap71-basic-s2i.json"
def projectDev = "${NAMESPACE}"
def projectQA = "${NAMESPACE}-staging"
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
		}/*
                stage('cleanup') {
                        openshift.withCluster(ocpClusterName) {
                                openshift.withProject(projectDev) {
                                    echo "[CleanUP]Using project: ${openshift.project()}"
                                    // delete everything with this template label
				    openshift.selector("all", [ template : "${templateName}" ]).delete()
                                    if (openshift.selector("secrets","${templateName}").exists()) {
                                        openshift.selector("secrets","${templateName}").delete()
                                    }
                                    if (openshift.selector("configMap","${NOMBRE_APP}-config").exists()) {
                                        openshift.selector("configMap","${NOMBRE_APP}-config").delete()
                                    }

                                }
                        }
                }*/
		stage('create') {
			openshift.withCluster(ocpClusterName) {
				openshift.withProject(projectDev) {
					echo '[CREATE]Ejecutando create'
					checkout scm
					def res
					echo '[CREATE]Proyecto no existe, lo creamos a partir del template'
					// echo "[CREATE] PARAMS ${params}"
					
					res = openshift.newApp( "${WORKSPACE}/"+templatePath )

					bc = res.narrow('bc')
					dc = res.narrow('dc')
				}
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
			}
		}
//-----------------------------------------------staging-ini---------------------------------------------------------	    	    
	    	stage('promote to QA') {
			openshift.withCluster(ocpClusterName) {
			    openshift.withProject(projectDev) {
    			  mail (
                    to: "$APPROVAL_EMAIL",
                    subject: "${NOMBRE_APP} (${env.BUILD_NUMBER}) está en espera de ser promovida a QA",
                    body: "Por favor verificar en: ${env.BUILD_URL}.");
                  timeout(10) {
                    input "Listo para promover a QA?"
                  }
			    }
			}
		}
		stage('cleanup QA') {
		  openshift.withCluster(ocpClusterName) {
			openshift.withProject(projectQA) {
				  echo '[CLEANUP]Ejecutando cleanup'
				  openshift.selector("all", [ template : templateName ]).delete()
				  if (openshift.selector("secrets", "gitsecret").exists()) {
					 openshift.selector("secrets", "gitsecret").delete()
					 echo '[CLEANUP]Secrets eliminados'
				  }
				  if (openshift.selector("configMap","${NOMBRE_APP}-config").exists()) {
						openshift.selector("configMap","${NOMBRE_APP}-config").delete()
				  }
			}
		  }
		}
		stage('Promoting object from Dev to QA') {
			openshift.withCluster(ocpClusterName) {
				echo '[PROMOTE-QA] Ejecutando copia de OBJ a QA'
				openshift.withProject( projectQA ) {
					if (!openshift.selector("dc/${NOMBRE_APP}").exists()) {
						echo '[PROMOTE-QA]Proyecto no existe, lo creamos a partir del template'
						res = openshift.newApp( "${WORKSPACE}/"+templatePath )
					}else{
						echo '[PROMOTE-QA] Proyecto existe, lo actualizamos'
						def template = openshift.withProject( 'openshift' ) {
							openshift.selector('template',templateName).object()
						}
						//WORKAROUND: No se puede modificar un DC que tanga el label kubectl.kubernetes.io/last-applied-configuration
						// Forzamos su eliminacion
						openshift.raw("annotate", "dc/${NOMBRE_APP}", "kubectl.kubernetes.io/last-applied-configuration-")
						res = openshift.apply(openshift.process( templateName ) );
					}
				}
			}
		}
		stage('tag into QA Namespace') {
		  openshift.withCluster(ocpClusterName) {
			//openshift.verbose()
			openshift.withProject(projectDev) {
			  echo '[TAG]Ejecutando tag'
			  openshift.tag("${projectDev}/${NOMBRE_APP}:latest", "${projectQA}/${NOMBRE_APP}:latest")
			}
			//openshift.verbose(false)
		  }
		}
		stage('patching QA imageStream And Configs') {
			openshift.withCluster(ocpClusterName) {
				openshift.withProject(projectQA) {
				  echo '[PATCH] Parchando QA: Cambiando env var from '
				}
			}
		}
		stage('deploy on QA') {
			openshift.withCluster(ocpClusterName) {
				//openshift.verbose()
				openshift.withProject(projectQA) {
					echo '[DEPLOY]Ejecutando deploy QA'
				    def rm = openshift.selector("dc/${NOMBRE_APP}").rollout().latest()
				    timeout(5) {
					  openshift.selector("dc/${NOMBRE_APP}").related('pods').untilEach(1) {
					    return (it.object().status.phase == "Running")
					  }
				    }
				}
			}
		}
//-----------------------------------------------staging-end---------------------------------------------------------
	}
} catch (Exception e ) {
    echo "Error: ${e}"
}
