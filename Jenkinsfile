properties(
  [
    pipelineTriggers(
      [
        [
          $class: 'CIBuildTrigger',
          checks: [],
          overrides: [topic: "Consumer.rh-jenkins-ci-plugin.cc2a6550-87f8-4030-8454-e221e27c48f7.VirtualTopic.qe.ci.>"],
          providerName: 'Red Hat UMB',
          selector: 'name = \'atomic-openshift\' AND CI_TYPE = \'brew-tag\' AND tag LIKE \'rhaos-%-rhel-%-newarches-candidate\''
        ]
      ]
    ),
    parameters(
      [
        string(
          defaultValue: 'x86_64,ppc64le',
          description: 'A comma separated list of architectures to run the test on. Valid values include [x86_64, ppc64le, aarch64, s390x].',
          name: 'ARCHES'
        ),
        string(
          defaultValue: 'SSHPRIVKEY',
          description: 'SSH private key Jenkins credential ID for Beaker/SSH operations.',
          name: 'SSHPRIVKEYCREDENTIALID'
        ),
        string(
          defaultValue: 'SSHPUBKEY',
          description: 'SSH public key Jenkins credential ID for Beaker/SSH operations.',
          name: 'SSHPUBKEYCREDENTIALID'
        ),
        string(
          defaultValue: 'KEYTAB',
          description: 'Kerberos keytab file Jenkins credential ID for Beaker/SSH operations.',
          name: 'KEYTABID'
        ),
        string(
          defaultValue: 'JENKINS_SLAVE_CREDENTIALS',
          description: 'Jenkins slave credential ID for connecting slaves using cinch via JSwarm.',
          name: 'JENKINSSLAVECREDENTIALID'
        ),
        string(
          defaultValue: 'https://github.com/RedHat-MultiArch-QE/multiarch-ci-libraries',
          description: 'Repo for shared libraries.',
          name: 'LIBRARIES_REPO'
        ),
        string(
          defaultValue: 'v0.2',
          description: 'Git reference to the branch or tag of shared libraries.',
          name: 'LIBRARIES_REF'
        ),
        string(
          defaultValue: '',
          description: 'Repo for tests to run. If left blank, the current repo is assumed (*note* this default will only work for multibranch pipelines).',
          name: 'TEST_REPO'
        ),
        string(
          defaultValue: '',
          description: 'Git reference to the branch or tag of the tests repo.',
          name: 'TEST_REF'
        ),
        string(
          defaultValue: 'tests',
          description: 'Directory containing tests to run. Should at least one of the follow: an ansible-playbooks directory containing one or more test directories each of which having a playbook.yml, a scripts directory containing one or more test directories each of which having a run-test.sh',
          name: 'TEST_DIR'
        ),
        string(
          defaultValue: '',
          description: 'Contains the CI_MESSAGE for a message bus triggered build.',
          name: 'CI_MESSAGE'
        ),
        string(
          name: 'ORIGIN_REPO',
          description: 'Origin repo',
          defaultValue: 'https://github.com/openshift/origin.git'
        ),
        string(
          name: 'ORIGIN_BRANCH',
          description: 'Origin branch',
          defaultValue: 'master'
        ),
        string(
          name: 'OS_BUILD_ENV_IMAGE',
          description: 'openshift-release image',
          defaultValue: 'openshiftmultiarch/origin-release:golang-1.8'
        )
      ]
    )
  ]
)

library(
  changelog: false,
  identifier: "multiarch-ci-libraries@${params.LIBRARIES_REF}",
  retriever: modernSCM([$class: 'GitSCMSource',remote: "${params.LIBRARIES_REPO}"])
)

List arches = params.ARCHES.tokenize(',')
def config = TestUtils.getProvisioningConfig(this)

TestUtils.runParallelMultiArchTest(
  this,
  arches,
  config,
  { host ->
    /*********************************************************/
    /* TEST BODY                                             */
    /* @param host               Provisioned host details.   */
    /*********************************************************/
    dir('test') {
      stage ('Download Test Files') {
        if (params.TEST_REPO) {
          git url: params.TEST_REPO, branch: params.TEST_REF, changelog: false
        }
        else {
          checkout scm
        }
      }

      def gopath = "${pwd(tmp: true)}/go"
      def failed_stages = []
      withEnv(["GOPATH=${gopath}", "PATH=${PATH}:${gopath}/bin"]) {
        stage('Prep') {
          sh 'git config --global user.email "jpoulin@redhat.com" && git config --global user.name "Jeremy Poulin"'
          git(url: params.ORIGIN_REPO, branch: params.ORIGIN_BRANCH)
          sh '''
          #!/bin/bash -xeu
          git remote add detiber https://github.com/detiber/origin.git || true
          git fetch detiber
          git merge detiber/multiarch
          '''
        }
        try {
          stage('Pre-release Tests') {
            sh '''
            #!/bin/bash -xeu
            hack/env JUNIT_REPORT=true DETECT_RACES=false TIMEOUT=300s make check -k
            '''
          }
        } catch (exc) {
          failed_stages+='Pre-release Tests'
          currentBuild.result = 'UNSTABLE'
        }
        stage('Locally build release') {
          try {
            sh '''
            #!/bin/bash -xeu
            hack/env hack/build-base-images.sh
            hack/env JUNIT_REPORT=true make release
            '''
          } catch (e) {
            currentBuild.result = 'FAILURE'
            throw e
          }
        }
        try {
          stage('Integration Tests') {
            sh '''
            #!/bin/bash -xeu
            hack/env JUNIT_REPORT='true' make test-tools test-integration
            '''
          }
        } catch (e) {
          failed_stages+='Integration Tests'
          currentBuild.result = 'UNSTABLE'
        }
        try {
          stage('End to End tests') {
            sh '''
            #!/bin/bash -xeu
            arch=$(go env GOHOSTARCH)
            OS_BUILD_ENV_PRESERVE=_output/local/bin/linux/${arch}/end-to-end.test hack/env make build-router-e2e-test
            OS_BUILD_ENV_PRESERVE=_output/local/bin/linux/${arch}/etcdhelper hack/env make build WHAT=tools/etcdhelper
            OPENSHIFT_SKIP_BUILD='true' JUNIT_REPORT='true' make test-end-to-end -o build
            '''
          }
        } catch (e) {
          failed_stages+='End to End Tests'
          currentBuild.result = 'UNSTABLE'
        }
      }
    } catch (e) {
      echo e
    } finally {
      stage ('Archive Test Output') {
        archiveArtifacts '_output/scripts/**/*'
        junit '_output/scripts/**/*.xml'
      }
    }

    /*****************************************************************/
    /* END TEST BODY                                                 */
    /* Do not edit beyond this point                                 */
    /*****************************************************************/
  },
  { Exception exception, def host ->
    echo "Exception ${exception} occured on ${host.arch}"
    if (host.arch.equals("x86_64") || host.arch.equals("ppc64le")) {
      currentBuild.result = 'FAILURE'
    }
  }
)
