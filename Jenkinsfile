@Library('pmd@family-pmd4') _

import uk.org.floop.jenkins_pmd.Drafter

pipeline {
    agent {
        label 'master'
    }
    stages {
        stage('Transform') {
            agent {
                docker {
                    image 'cloudfluff/databaker'
                    reuseNode true
                }
            }
            steps {
                sh 'jupyter-nbconvert --to python --stdout orgs_to_turtle.ipynb | python'
            }
        }
        stage('Publish results') {
            steps {
                script {
                    def pmd = pmdConfig('pmd')
                    for (myDraft in pmd.drafter
                            .listDraftsets(Drafter.Include.OWNED)
                            .findAll { it['display-name'] == env.JOB_NAME }) {
                        pmd.drafter.deleteDraftset(myDraft.id)
                    }
                    def id = pmd.drafter.createDraftset(env.JOB_NAME).id
                    for (graph in util.jobGraphs(pmd, id)) {
                        pmd.drafter.deleteGraph(id, graph)
                        echo "Removing own graph ${graph}"
                    }
                    String graph = "http://gss-data.org.uk/graph/organizations"
                    echo "Adding gov orgs as ${graph}"
                    pmd.drafter.addData(id, "${WORKSPACE}/gov_orgs.ttl", "text/turtle", "UTF-8", graph)
                    pmd.drafter.publishDraftset(id)
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts '*.ttl'
        }
    }
}
