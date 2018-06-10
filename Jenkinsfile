node {

    checkout scm

    def port = 5085
    def mode = "stage"

    stage ("Build and run docker image") {

        shortCommit = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()

        echo "Building $env.BUILD_ID from $shortCommit"

        //sh "./etc/scripts/redeploy-staging.sh"

        sh '''
            set -e

            WORKSPACE=$(pwd)

            echo "Work dir $WORKSPACE"

            docker-compose down || true

            docker rm webrtcshowcase_nginx_1 || true

            docker rmi -f webrtcshowcase_nginx || true

            cp -R etc/certs/webrtc-showcase.trembit.com/* "$WORKSPACE/nginx/certs/"

            cd "$WORKSPACE/web"

            #rm -Rf dist

            docker build -t webrtc-showcase/web .

            docker rm webrtc-showcase-web || true

            docker run --name webrtc-showcase-web -t webrtc-showcase/web ng build --prod --deploy-url /static --base-href https://webrtc-showcase.trembit.com:5085/static/

            mkdir -p dist

            # EPIC: it works, but it throws "operation not permitted"
            # https://github.com/moby/moby/issues/3986
            docker cp webrtc-showcase-web:/usr/src/app/dist "$WORKSPACE/web" || true

            # --rm flag doesn't leave container available. So we manually remove container
            docker rm webrtc-showcase-web

            cd "$WORKSPACE"

            docker-compose build

            docker-compose up -d
        '''
    }
}
