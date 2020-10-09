import org.jenkinsci.plugins.workflow.steps.FlowInterruptedException

stage('root') {
  try {
    parallel w2016: {
      stage('2016') {
        node('packer') {
           sleep 1
           executePackerType('2016')
        }
      }
    }
  } catch (e) {
    currentBuild.result = 'FAILURE'
    notifyFailed()
    throw e
  } finally {
    println 'currentBuild.result: ' + currentBuild.result
    if (currentBuild.result != 'FAILURE') {
      notifySuccessful()
    }
  }
}

def executePackerType(String host_type) {
  checkout([$class: 'GitSCM', branches: [[name: env.BRANCH_NAME]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'packer-windoze']],                submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'f37e5623-35de-4909-ba30-72a3dad7a582', url: 'git@github.com:jpmat296/packer-windoze.git']]])
  stage('packer') {
    timestamps {
      try {
        stage('package') {
          sh """#!/bin/bash
            set -xe
            bash ~/bin/vms_destroy.sh || true
            source /usr/local/pyenv/.pyenvrc
            cd packer-windoze
            pipenv install
            export PATH=\$(pipenv --venv)/bin:\$PATH
            hash -r
            ansible-playbook packer-setup.yml -e man_packer_setup_host_type=$host_type \
                -e opt_packer_setup_disk_size_mib=204800
            packer fix $host_type/packer.json > tmp.json && mv tmp.json $host_type/packer.json
            packer build -force $host_type/packer.json
          """
        }
      } finally {
        stage('destroy') {
          sh """#!/bin/bash
            set -xe
            bash ~/bin/vms_destroy.sh || true
          """
        }
      }
    }
  }
}

def notifySuccessful() {
  emailext (
      subject: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",

body: """<p>SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
      to: 'jp@matsusoft.com'
  )
}

def notifyFailed() {
  emailext (
      subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
      to: 'jp@matsusoft.com'
  )
}
