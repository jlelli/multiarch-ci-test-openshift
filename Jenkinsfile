properties([
  parameters([
    choiceParam(
      name: 'ARCH',
      choices: "x86_64\nppc64le\naarch64\ns390x",
      description: 'Architecture'
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
      // defaultValue: 'ppc64le-rebase-wip'
    ),
    string(
      name: 'OS_BUILD_ENV_IMAGE',
      description: 'openshift-release image',
      defaultValue: 'openshiftmultiarch/origin-release:golang-1.8'
    )
  ])
])

def provisionedNode = null
def provisionedNodeBuildNumber = null

ansiColor('xterm') {
  timestamps {
    node('master') {
      try {
        try {
          stage('Provision Slave') {
            def buildResult = build([
              job: 'provision-multiarch-slave',
              parameters: [
                string(name: 'ARCH', value: arch),
              ],
              propagate: true,
              wait: true
            ])

            provisionedNodeBuildNumber = buildResult.getNumber().toString()

            // Get results of provisioning job
            step([$class: 'CopyArtifact', filter: 'slave.properties',
              projectName: 'provision-multiarch-slave',
              selector: [
                $class: 'SpecificBuildSelector',
                buildNumber: provisionedNodeBuildNumber
              ]
            ])

            // Load slave properties (you may need to turn off sandbox or approve this in Jenkins)
            def slaveProps = readProperties file: 'slave.properties'

            // Assign the appropriate slave name
            provisionedNode = slaveProps.name
          }
        } catch (e) {
          println e
          provisionedNodeBuildNumber = ((e =~ "(provision-multiarch-slave #)([0-9]*)")[0][2])
          currentBuild.result = 'FAILURE'
        }

        try {
          node(provisionedNode) {
            // no op
          }
        } catch (e) {
          println(e)
        }
      } finally {
        stage ('Teardown Slave') {
          build([job: 'teardown-multiarch-slave',
            parameters: [
              string(name: 'BUILD_NUMBER', value: provisionedNodeBuildNumber)
            ],
            propagate: true,
            wait: true
          ])
        }
      }
    }
  }
}
