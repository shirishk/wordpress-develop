def label = "mypod-${UUID.randomUUID().toString()}"
// Create pod template using desired image.
podTemplate(label: label, containers: [
        containerTemplate(name: 'jnlp', image: '10.21.236.86:5000/jnlp-slave-sudo', alwaysPullImage: true, args: '${computer.jnlpmac} ${computer.name}', workingDir: '/home/jenkins/dev-jenkins/'+env.BUILD_NUMBER),
        containerTemplate(name: 'buildwordpress', image: '10.21.236.86:5000/build-wordpress', workingDir: '/home/jenkins/dev-jenkins/'+env.BUILD_NUMBER, ttyEnabled: true, command: 'cat')],
    volumes: [
        persistentVolumeClaim(mountPath: '/home/jenkins/dev-jenkins', claimName: 'dev-jenkins', readOnly: false)],
    imagePullSecrets: [ 'myregistrykey' ]) 
    
    {
    node(label) {
    container('buildwordpress') {
            // Git clone source code.
            stage('Clone Repository') {
                git branch: 'develop', credentialsId: '4063a731-7121-487c-918c-93c2f103d1c7', url: 'http://10.21.236.87:8080/root/wordpress-develop.git'
            }
            // Install dependancies and build project.
            stage('Build project') {
                sh "npm install"
            }
            // Run QUnit test cases
            stage('Run Unit Tests') {
                sh "grunt qunit:reports --force"
            }
            // Create tar.gz package
            stage('Create Package') {
                sh "mkdir wordpress" 
                sh "cp -rf build/* wordpress/"
                sh "tar -czvf Wordpress.tar.gz wordpress/"
            }
            stage('Add to Nexus artifacts') {
                nexusArtifactUploader artifacts: [[artifactId: 'wordpress-artifacts', classifier: 'wordpress-artifacts', file: 'Wordpress.tar.gz', type: 'tar.gz']], credentialsId: 'd47d3a3d-83dc-440d-9bca-31e14062ae40', groupId: 'wp_gid', nexusUrl: '10.21.236.86', nexusVersion: 'nexus3', protocol: 'http', repository: 'wordpress-artifacts', version: '1.0.0.'+env.BUILD_NUMBER
            }
        }
    }
}
