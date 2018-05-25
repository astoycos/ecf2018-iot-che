# Installation instructions

## Install master

* Create new machine
  * CentOS 7
  * Type: CX41+
  * User Data: [user-data.yml](user-data.yml)
  * Add SSH keys
* Assign floating IP
* Renew Let's encrypt certificate
* Setup OpenShift
  * Run installer … see [openshift/README.md](openshift/README.md) 

## Installing EnMasse

Download release:

    curl -LO https://github.com/EnMasseProject/enmasse/releases/download/0.20.0/enmasse-0.20.0.tgz
    tar xzf enmasse-0.20.0.tgz

Deploy EnMasse:

    oc new-project enmasse --display-name='EnMasse Instance'
    #oc create secret tls api-server-cert --cert=/home/jreimann/letsencrypt/fantastic.iot-playground.org/fullchain.cer --key=/home/jreimann/letsencrypt/fantastic.iot-playground.org/fantastic.iot-playground.org.key
    #oc create configmap address-space-controller-config --from-literal=wildcardEndpointCertSecret=api-server-cert
    
    # enmasse-0.20.0-rc3/deploy.sh: comment out "oc create configmap address-space-controller-config"
    # enmasse-0.20.0-rc3/deploy.sh: comment out "create_self_signed_cert … api-server-cert"
    
    ./enmasse-0.20.0/deploy.sh -n enmasse -u iot -m https://fantastic.iot-playground.org:8443 -k /home/jreimann/letsencrypt/fantastic.iot-playground.org/fantastic.iot-playground.org.key -c /home/jreimann/letsencrypt/fantastic.iot-playground.org/fullchain.cer

And configure it:

    oc create -f enmasse/addresses.yml

    curl -X POST -T "enmasse/addresses.json" -H "content-type: application/json" https://$(oc -n enmasse get route restapi -o jsonpath='{.spec.host}')/apis/enmasse.io/v1alpha1/namespaces/enmasse/addressspaces/default/addresses

## Strimzi / Kafka

On the master:

    git clone -b release-0.4.x https://github.com/strimzi/strimzi
    oc new-project strimzi --display-name='Strimzi / Kafka'
    oc create -f strimzi/examples/install/cluster-operator
    oc create -f strimzi/examples/templates/cluster-operator

Deploy a Kafka cluster and topics:

    oc project strimzi
    oc create -f kafka

## Installing Eclipse Hono

Deploy Hono:

    oc new-project hono --display-name='Eclipse Hono™'
    
    cd hono/example/src/main/deploy/openshift_s2i
    oc create configmap influxdb-config --from-file="../influxdb.conf"
    
    oc process -f hono-template.yml \
      -p "ENMASSE_NAMESPACE=enmasse" \
      -p "GIT_BRANCH=0.6.x"| oc create -f -
    
    oc env dc/hono-service-device-registry HONO_REGISTRY_SVC_MAX_DEVICES_PER_TENANT=10000

## Install simulation

    oc new-project iot-simulator --display-name='IoT workload simulator'
    oc process -f hono-demo-1/src/openshift/demo.yml | oc create -f -
    oc scale --replicas=0 dc hono-demo-1-consumer-influxdb

## Install Grafana

    oc new-project grafana --display-name='Grafana Dashboard'
    oc process -f hono/example/src/main/deploy/openshift/grafana-template.yml \
      -p ADMIN_PASSWORD=admin | oc create -f -
    ./hono/example/src/main/deploy/configure_grafana.sh "$(oc get route grafana -o jsonpath='{.spec.host}')" 443 https
    oc apply -f https/grafana.yml
    
    GRAFANA_URL="$(oc -n grafana get route grafana --template='{{ .spec.host }}')"
    curl -X POST -T grafana/ds_payload.json -H "content-type: application/json" "https://admin:admin@$GRAFANA_URL/api/datasources"
    curl -X POST -T grafana/dashboard_payload.json -H "content-type: application/json" "https://admin:admin@$GRAFANA_URL/api/dashboards/db"

## Deploy bridge

    oc new-project hono-consumer --display-name 'Hono Consumer'
    
    oc new-app fabric8/s2i-java~https://github.com/ctron/hono-example-bridge.git
    #oc patch dc/hono-example-bridge --patch '{"spec":{"template":{"spec":{"containers":[{"name":"hono-example-bridge", "ports":[{"containerPort":"8778", "name":"jolokia" }]}]}}}}'
    oc set probe dc/hono-example-bridge --readiness --liveness --get-url=http://:8080/health

Set up GitHub webhook:

    # Register GitHub web hook

## Install Eclipse Che

From the root of this repository run:

    cd che/deploy/openshift/templates
    oc new-project che --display-name='Eclipse Che™'
    
    oc new-app -f multi/postgres-template.yaml -p CHE_VERSION=6.5.0
    oc new-app -f multi/keycloak-template.yaml -p CHE_VERSION=6.5.0 -p KEYCLOAK_PASSWORD=<initial-admin-password> -p ROUTING_SUFFIX=fantastic.iot-playground.org -p PROTOCOL=https
    oc apply -f pvc/che-server-pvc.yaml
    oc new-app -f che-server-template.yaml -p CHE_VERSION=6.5.0 -p ROUTING_SUFFIX=fantastic.iot-playground.org -p CHE_MULTIUSER=true -p PROTOCOL=https -p WS_PROTOCOL=wss -p TLS=true
    oc set volume dc/che --add -m /data --name=che-data-volume --claim-name=che-data-volume
    oc apply -f https

Once Che is up and running:

    CHE_URL="https://$(oc -n che get route che --template='{{ .spec.host }}')"
    
    echo "Open browser at: $CHE_URL/f?url=https://github.com/ctron/hono-example-bridge"
    echo "Open browser at: $CHE_URL/f?url=https://github.com/ctron/hono-example-demo-gauge"

## Deploy demo as project

    oc new-project demo-gauge
    oc new-app centos/nodejs-8-centos7~https://github.com/ctron/hono-example-demo-gauge
    oc apply -f https/demo-gauge.yml

Set up GitHub webhook:

    # Register GitHub web hook
