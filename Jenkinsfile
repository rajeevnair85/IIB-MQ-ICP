podTemplate(label: 'iibpod',
    volumes: [
        hostPathVolume(hostPath: '/etc/docker/certs.d', mountPath: '/etc/docker/certs.d'),
        hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),
        secretVolume(secretName: 'registry-account', mountPath: '/var/run/secrets/registry-account'),
        configMapVolume(configMapName: 'registry-config', mountPath: '/var/run/configs/registry-config')
    ],
    containers: [
        containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'docker' , image: 'docker:17.06.1-ce', ttyEnabled: true, command: 'cat')
  ]) {

    node('iibpod') {
        checkout scm
        container('docker') {
            stage('Build Docker Image') {
                sh """
                #!/bin/bash
			
                NAMESPACE=`cat /var/run/configs/registry-config/namespace`
                REGISTRY=`cat /var/run/configs/registry-config/registry`
			

                docker build -t \${REGISTRY}/\${NAMESPACE}/iib10-toolkit-mq .
                """
            }
            stage('Push Docker Image to Registry') {
                sh """
                #!/bin/bash
                NAMESPACE=`cat /var/run/configs/registry-config/namespace`
                REGISTRY=`cat /var/run/configs/registry-config/registry`

                set +x
                DOCKER_USER=`cat /var/run/secrets/registry-account/username`
                DOCKER_PASSWORD=`cat /var/run/secrets/registry-account/password`
                docker login -u=\${DOCKER_USER} -p=\${DOCKER_PASSWORD} \${REGISTRY}
                set -x

                docker push \${REGISTRY}/\${NAMESPACE}/iib10-toolkit-mq
                """
            }
        }
        container('kubectl') {
            stage('Deploy new Docker Image') {
                sh """
                #!/bin/bash
                set +e
                NAMESPACE=`cat /var/run/configs/registry-config/namespace`
                REGISTRY=`cat /var/run/configs/registry-config/registry`
                DEPLOYMENT=`kubectl get deployments -l app=iibmq1 -o name`

                kubectl get \${DEPLOYMENT}

                if [ \${?} -ne "0" ]; then
                    # No deployment to update
                    echo 'No deployment to update'
                    #exit 1
		    kubectl create -f iib-mq.yml
		    kubectl create -f service.yml
		    exit 0
                fi

                # Update Deployment
                #kubectl set image \${DEPLOYMENT} web=\${REGISTRY}/\${NAMESPACE}/iib10-toolkit-mq:${env.BUILD_NUMBER}
		kubectl set image \${DEPLOYMENT} iibmq1=\${REGISTRY}/\${NAMESPACE}/iib10-toolkit-mq
                kubectl rollout status \${DEPLOYMENT}
                """
            }
        }
    }
}
