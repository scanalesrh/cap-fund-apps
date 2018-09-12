NAMESPACE="myproject"
NOMBRE_APP="eap-app"
APPROVAL_EMAIL=""

def templateName = "eap71-basic-s2i"
def templateFilePre = "eap71-basic-s2i-pre.json"
def templateFileProd = "eap71-basic-s2i-prod.json"
def appEnvFile = "eap-app.env"
def projectPre = "${NAMESPACE}"
def projectProd = "${NAMESPACE}-prod"
def ocpClusterName = "master"
def bc = ""
def dc = ""
try {
    timeout(20) {
		stage('preamble') {
			openshift.withCluster(ocpClusterName) {
				openshift.withProject(projectPre) {
					echo "[PREAMBLE]Using project: ${openshift.project()}"
				}
			}
		}
                stage('cleanup') {
                        openshift.withCluster(ocpClusterName) {
                                openshift.withProject(projectPre) {
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
                }
		stage('create') {
			openshift.withCluster(ocpClusterName) {
				openshift.withProject(projectPre) {
					echo '[CREATE]Ejecutando create'
					checkout scm
					def res
					echo '[CREATE]Proyecto no existe, lo creamos a partir del template'
					// echo "[CREATE] PARAMS ${params}"
					
					res = openshift.newApp( "-f","${WORKSPACE}/"+templateFilePre )

					bc = res.narrow('bc')
					dc = res.narrow('dc')
				}
			}
		}
		stage('build') {
			openshift.withCluster(ocpClusterName) {
				//openshift.verbose()
				openshift.withProject(projectPre) {
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
				openshift.withProject(projectPre) {
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
	    	stage('promote to Prod') {
			openshift.withCluster(ocpClusterName) {
			    openshift.withProject(projectPre) {
    			/*  mail (
                    to: "$APPROVAL_EMAIL",
                    subject: "${NOMBRE_APP} (${env.BUILD_NUMBER}) está en espera de ser promovida a Prod",
                    body: "Por favor verificar en: ${env.BUILD_URL}.");*/
                  timeout(10) {
                    input "Listo para promover a Prod?"
                  }
			    }
			}
		}
		stage('cleanup Prod') {
		  openshift.withCluster(ocpClusterName) {
			openshift.withProject(projectProd) {
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
		stage('Promoting object from Pre to Prod') {
			openshift.withCluster(ocpClusterName) {
				echo '[PROMOTE-Prod] Ejecutando copia de OBJ a Prod'
				openshift.withProject( projectProd ) {
					if (!openshift.selector("dc/${NOMBRE_APP}").exists()) {
						echo '[PROMOTE-Prod]Proyecto no existe, lo creamos a partir del template'
						res = openshift.newApp( "${WORKSPACE}/"+templateFileProd )
					}else{
						echo '[PROMOTE-Prod] Proyecto existe, lo actualizamos'
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
		stage('tag into Prod Namespace') {
		  openshift.withCluster(ocpClusterName) {
			//openshift.verbose()
			openshift.withProject(projectPre) {
			  echo '[TAG]Ejecutando tag'
			  openshift.tag("${projectPre}/${NOMBRE_APP}:latest", "${projectProd}/${NOMBRE_APP}:latest")
			}
			//openshift.verbose(false)
		  }
		}
		stage('patching Prod imageStream And Configs') {
			openshift.withCluster(ocpClusterName) {
				openshift.withProject(projectProd) {
				  echo '[PATCH] Parchando Prod: Cambiando env var from '
				}
			}
		}
		stage('deploy on Prod') {
			openshift.withCluster(ocpClusterName) {
				//openshift.verbose()
				openshift.withProject(projectProd) {
					echo '[DEPLOY]Ejecutando deploy Prod'
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
