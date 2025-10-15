def appname   = "hello-newapp"
def repo      = "Cheeza42"  // Docker Hub username
def artifactory = "docker.io"
def appimage  = "docker.io/${repo}/${appname}"
def apptag    = "${env.BUILD_NUMBER}"

// NOTE: Ensure this pod runs in the same namespace where the ConfigMap exists.
// If your agents run in a non-default namespace, add: podTemplate(namespace: '<your-ns>', ...)
podTemplate(
  containers: [
    containerTemplate(name: 'jnlp',   image: 'jenkins/inbound-agent', ttyEnabled: true),
    containerTemplate(
      name: 'kaniko',
      image: 'gcr.io/kaniko-project/executor:latest',
      command: '/busybox/cat',
      ttyEnabled: true
    )
  ],
  volumes: [
    // Mount the ConfigMap that contains config.json to /kaniko/.docker/
    configMapVolume(mountPath: '/kaniko/.docker/', configMapName: 'kaniko-config'),
    // Optional cache for faster builds
    emptyDirVolume(mountPath: '/kaniko/.cache')
  ]
) {
  node(POD_LABEL) {
    stage('checkout') {
      container('jnlp') {
        sh '/usr/bin/git config --global http.sslVerify false'
        checkout scm
      }
    }

    stage('build & push (kaniko)') {
      container('kaniko') {
        echo "Building & pushing image with Kaniko..."
        sh """
          /kaniko/executor \
            --context "${WORKSPACE}" \
            --dockerfile "${WORKSPACE}/Dockerfile" \
            --destination "${appimage}:${apptag}" \
            --snapshotMode=redo \
            --verbosity=info
        """
      }
    }

    // stage('deploy') {
    //   container('kaniko') {
    //     echo "Deployment step (optional)..."
    //   }
    // }
  }
}
