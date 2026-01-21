pipeline {
  agent any

  tools {
    nodejs "Node25"
    dockerTool 'Dockertool'
  }

  environment {
    IMAGE_NAME = "hola-mundo-node:latest"
    BASE_PORT = "3000"
    // host_port = 3000 + (BUILD_NUMBER % 1000) => 3001..3999
    HOST_PORT = "${3000 + (env.BUILD_NUMBER as Integer) % 1000}"
    CONTAINER_PORT = "3000"

    // Si quieres evitar conflicto por nombre también:
    CONTAINER_NAME = "hola-mundo-node-${env.BUILD_NUMBER}"
  }

  stages {
    stage('Verificar herramientas') {
      steps {
        sh '''
          node -v
          npm -v
          docker --version
          docker info >/dev/null
        '''
      }
    }

    stage('Construir Imagen Docker') {
      steps {
        sh "docker build -t ${IMAGE_NAME} ."
      }
    }

    stage('Ejecutar Contenedor Node.js') {
      steps {
        sh '''
          set -e
          echo "Usando puerto: $HOST_PORT -> $CONTAINER_PORT"
          echo "Contenedor: $CONTAINER_NAME"

          # (Opcional) borra contenedor previo con el mismo nombre (por si reintentas el build)
          docker stop "$CONTAINER_NAME" || true
          docker rm "$CONTAINER_NAME" || true

          docker run -d --name "$CONTAINER_NAME" -p "$HOST_PORT:$CONTAINER_PORT" "$IMAGE_NAME"

          echo "URL: http://localhost:$HOST_PORT"
          sleep 2
          curl -sSf "http://localhost:$HOST_PORT/" | head -c 200 || true
          echo ""
        '''
      }
    }
  }

  post {
    always {
      sh '''
        echo "Contenedores relacionados:"
        docker ps --filter "name=hola-mundo-node" --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}" || true
      '''
    }
  }
}
