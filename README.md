# Multi-Arch CI Test OpenShift
This project serves as an example of how to leverage the [Multi-Arch CI Test Template](https://github.com/redhat-multiarch-qe/multiarch-ci-test-template) to run OpenShift Origin tests in RedHat's downstream CI environment. This test further expands on the template by serving as an example of how to parse RedHat CI messages and trigger based on package builds. Unfortunately, the test template can be run only on Jenkins enviroments deployed with the tools provided by the [Multi-Arch CI Provisioner](https://github.com/RedHat-MultiArch-QE/multiarch-ci-provisioner) which has limited support for upstream solutions. However, efforts are being made to support further multi-arch testing upstream as part of the great CentOS CI initiative (see [multiarch-openshift-ci](https://github.com/CentOS-PaaS-SIG/multiarch-openshift-ci) for an example of this effort specifically for OpenShift). You can see our latest template release notes [here](https://github.com/RedHat-MultiArch-QE/multiarch-ci-test-template/releases).

## Table of Contents
- [Getting Started](#getting-started)
- [License](#license)
- [Authors](#authors)

## Getting Started
For directions on how to create your own multi-arch tests, please visit the template [wiki](https://github.com/RedHat-MultiArch-QE/multiarch-ci-test-template/wiki).

## License
This project is licensed under the Apache 2.0 License - see the LICENSE file for details.

## Authors
This project would not be possible without the work of following people.
- [jlelli](https://github.com/jlelli/) - *Recovered and developed the base OpenShift test. Added features such as parsing for RedHat CI messages, input sanitation, and more.*
- [jaypoulz](https://github.com/jaypoulz/) - *Develops and maintains the current test template.*
- [detiber](https://github.com/detiber/) - *Engineered the starting point for this test in [multiarch-openshift-ci](https://github.com/CentOS-PaaS-SIG/multiarch-openshift-ci).*
- [arilivigni](https://github.com/arilivigni) - *Provided basis of the Jenkinsfile via [ci-pipeline](https://github.com/CentOS-PaaS-SIG/ci-pipeline).*
