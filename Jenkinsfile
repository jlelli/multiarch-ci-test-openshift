properties(
  [
    pipelineTriggers(
      [
        [
          $class: 'CIBuildTrigger',
          checks: [],
          overrides: [topic: "Consumer.rh-jenkins-ci-plugin.cc2a6550-87f8-4030-8454-e221e27c48f7.VirtualTopic.qe.ci.>"],
          providerName: 'Red Hat UMB',
          selector: 'name = \'atomic-openshift\' AND CI_TYPE = \'brew-tag\' AND tag LIKE \'rhaos-3.9-rhel-%-candidate\''
        ]
      ]
    ),
    parameters(
      [
        string(
          defaultValue: 'ppc64le',
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
          defaultValue: 'https://github.com/jlelli/multiarch-ci-libraries',
          description: 'Repo for shared libraries.',
          name: 'LIBRARIES_REPO'
        ),
        string(
          defaultValue: 'master',
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
        ),
        string(
          defaultValue: '',
          description: 'Build NVR for which to run the pipeline',
          name: 'BUILD_NVR'
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
    try {
      stage ("Install dependencies") {
        sh """
          sudo yum install -y yum-utils
          sudo yum-config-manager --add-repo https://download.fedoraproject.org/pub/epel/7/${host.arch};
          sudo yum-config-manager --add-repo http://download.devel.redhat.com/rel-eng/RCMTOOLS/rcm-tools-rhel-7-server.repo;
          sudo yum-config-manager --add-repo http://download-node-02.eng.bos.redhat.com/composes/nightly/EXTRAS-RHEL-7.4/latest-EXTRAS-7-RHEL-7/compose/Server/${host.arch}/os;
          sudo yum-config-manager --add-repo http://download-node-02.eng.bos.redhat.com/rel-eng/updates/RHEL-7.4/latest-RHEL-7/compose/Server/${host.arch}/os/;
          sudo yum-config-manager --add-repo http://download-node-02.eng.bos.redhat.com/nightly/EXTRAS-RHEL-7.4/latest-EXTRAS-7-RHEL-7/compose/Server/${host.arch}/os/;
          sudo yum-config-manager --add-repo http://download-node-02.eng.bos.redhat.com/rel-eng/updates/RHEL-7.4/latest-RHEL-7/compose/Server-optional/${host.arch}/os/;
          sudo yum-config-manager --add-repo http://download-node-02.eng.bos.redhat.com/rcm-guest/puddles/OpenStack/12.0-RHEL-7/latest/RH7-RHOS-12.0/${host.arch}/os/;
          sudo rpm --import http://download.eng.bos.redhat.com/composes/nightly/EXTRAS-RHEL-7.4/latest-EXTRAS-7-RHEL-7/compose/Server/${host.arch}/os/RPM-GPG-KEY-redhat-beta;
          sudo rpm --import http://download.eng.bos.redhat.com/composes/nightly/EXTRAS-RHEL-7.4/latest-EXTRAS-7-RHEL-7/compose/Server/${host.arch}/os/RPM-GPG-KEY-redhat-release;
          sudo rpm --import https://getfedora.org/static/352C64E5.txt;
          sudo yum install -y bc git make golang docker jq bind-utils koji brewkoji openvswitch;
          echo "[registries.search]" | sudo tee /etc/containers/registries.conf >/dev/null;
          echo "registries = ['registry.access.redhat.com']" | sudo tee --append /etc/containers/registries.conf >/dev/null;
          echo "" | sudo tee --append /etc/containers/registries.conf >/dev/null;
          echo "[registries.insecure]" | sudo tee --append /etc/containers/registries.conf >/dev/null;
          echo "registries = ['172.30.0.0/16']" | sudo tee --append /etc/containers/registries.conf >/dev/null;
          echo "" | sudo tee --append /etc/containers/registries.conf >/dev/null;
          echo "[registries.block]" | sudo tee --append /etc/containers/registries.conf >/dev/null;
          echo "registries = []" | sudo tee --append /etc/containers/registries.conf >/dev/null;
          sudo rm -rf /var/lib/docker;
          echo 'STORAGE_DRIVER=overlay2' | sudo tee /etc/sysconfig/docker-storage-setup > /dev/null;
          sudo systemctl enable docker;
          sudo systemctl start docker;
          sudo docker info
          env | sort
        """
        if (env.CI_MESSAGE != '') {
          getRPMFromCIMessage(env.CI_MESSAGE, host.arch)
          try {
            sh """
              ls *.rpm
              sudo yum --nogpgcheck localinstall -y *.rpm
            """
          } catch (exc) {
            println "No brew build packages found to install."
          }
        } else if (params.BUILD_NVR != '') {
          try {
            sh """
              #!/bin/sh -e
              RPMS=\$(brew buildinfo ${params.BUILD_NVR} | grep ${host.arch} | cut -d '/' -f10);
              for p in \${RPMS}; do echo \${p}; brew download-build --rpm \${p} >/dev/null; done;
              ls *.rpm;
              sudo yum --nogpgcheck localinstall -y *.rpm;
            """
          } catch (exc) {
            println "No brew build packages found for NVR ${params.BUILD_NVR}."
          }
        }
      }
      stage ("Start Cluster") {
        sh """
          sudo oc cluster up --image=docker-registry.engineering.redhat.com/multi-arch/ppc64le-openshift3-ose --version='for-test-3.9'
        """
      }

      def failed_stages = []
      try {
        stage ("End to End Tests") {
          sh """
            mkdir _out
            sudo KUBECONFIG=/var/lib/origin/openshift.local.config/master/admin.kubeconfig TEST_REPORT_DIR=./_out /usr/libexec/atomic-openshift/extended.test --ginkgo.v=true --ginkgo.skip="Prometheus|Serial|Flaky|Disruptive|Slow|should be applied to XFS filesystem when a pod is created" --ginkgo.focus="EmptyDir"
          """
        }
      } catch (e) {
        failed_stages+='End to End Tests'
        currentBuild.result = 'UNSTABLE'
      }
    } catch (e) {
      println(e)
    } finally {
      stage ('Archive Test Output') {
        archiveArtifacts '_out/*'
        junit '_out/*.xml'
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
