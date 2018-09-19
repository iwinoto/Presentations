# Scripts to run Istio demo

References:
* [IBM Cloud tutorial](https://console.bluemix.net/docs/containers/cs_tutorials_istio.html#istio_tutorial)

## Scenario
1. Load Istio from [IBM Helm chart](https://console.bluemix.net/containers-kubernetes/solutions/helm-charts/ibm/ibm-istio).
2. Show deployments
3. Run Bookinfo demo

## Cleanup
1. Delete the cluster
    ```
    $ ibmcloud ks cluster-rm <Cluster name>
    ```

## Setup
**PERFORM AT LEAST 3 HOURS BEFORE DEMO!!**

**This step involves IKS cluster creation which typically takes 30 minutes but this is dependent on available resources. It can take up to an hour and if there is a control plane issue, it will take longer than an hour.**

1. [Install `ibmcloud`](https://console.bluemix.net/docs/cli/reference/ibmcloud/download_cli.html#install_use) or make sure `ibmcloud` and plugins is up to date
    ```
    # Update CLI
    $ ibmcloud update
    ```
2. Install required plugins
    ```
    # Install required plugins
    $ ibmcloud plugin install IBM-Containers -r Bluemix
    $ ibmcloud plugin install container-registry -r Bluemix
    ```
3. Update installed plugins
    ```
    $ ibmcloud plugin update
    ```
4. Install `helm` or upgrade `helm` client
    ```
    # On OSX using homebrew
    # Install helm
    $ brew install kubernetes-helm
    # Upgrade helm
    $ brew upgrade kubernetes-helm
    ```
4. Login to IBM Cloud
5. Set IKS region
    ```
    $ ibmcloud region-set au-syd
    ```
6. Create IKS cluster
    1. For the desired zone, check if any vlans are available
        ```
        # get list of zones for current region
        $ ibmcloud ks zones
        $ ibmcloud ks vlans --zone syd01
        ```
    2. Check the `cluster-create.yaml` file. Make sure there are valid VLANs specified.
    2. Create the cluster, specify VLANs if existing VLAN should be used
        ```
        ibmcloud ks cluster-create --file cluster-create.yaml
        ibmcloud ks cluster-create --zone syd01 \
          --machine-type u2c.2x4 --hardware shared \
          --disable-disk-encrypt \
          --workers 2 --name iw-iks-demo \
          --public-vlan <public_VLAN_ID> \
          --private-vlan <private_VLAN_ID>
        ```
    3. Monitor cluster creation until *State* is `normal`.
        ```
        # List all clusters
        $ ibmcloud ks clusters
        # Get cluster details
        $ ibmcloud ks cluster-get <cluste name>
        ```
7. Get cluster context
    ```
    $ ibmcloud ks cluster-config <cluster name>
    $ export KUBECONFIG=<Cluster context path>
    ```
8. Initialise `helm` and install `tiller` on cluster
    ```
    # Use --upgrade to ensure client and server are both on the same version.
    $ helm init --upgrade
    ```
9. Check that `helm` installation is competed and ready. It can take a minute for the Tiller installation to complete, so try this a couple of times before worrying.
    ```
    $ helm version
    ```
10. Add the IBM helm repo
    ```
    $ helm repo add ibm-charts https://registry.bluemix.net/helm/ibm-charts
    ```
11. **DO NOT** add kube Custom Resource Definitions (CRDs) to the cluster. This seems to be part of the chart now.
12. Install Istio with helm
    ```
    # Create namespace
    kubectl create ns istio-system
    # Create kiali secret (username=admin, passphrase=password)
    kubectl apply -f kiali-secret.yaml
    # Install istio (with using values from istio-values.yaml)
    helm install ibm-charts/ibm-istio --name istio --namespace istio-system -f istio-values.yaml
    ```
13. Set NodePort type service for kiali
    ```
    kubectl -n istio-system expose service/kiali --type="NodePort" --name="kiali-port"
    # Get port
    kubectl -n istio-system get svc kiali-port
    # Get IKS public endpoint
    ibmcloud ks workers iw-iks-demo
    ```

### TODO
* [Configure logging](https://console.bluemix.net/docs/containers/cs_health.html#health)
    ```
    # set CF target for Org & Space
    $ ibmcloud target --cf-api https://api.au-syd.bluemix.net  -o iwinoto@au1.ibm.com -s demo
    # Default logging - container level, ibm log server
    $ ibmcloud ks logging-config-create iw-iks-demo
    ```

## Presentation
1. Overview of microservices mesh - what problems does it solve
2. Overview of Istio

## Demo
[IBM Cloud tutorial on Istio](https://console.bluemix.net/docs/containers/cs_tutorials_istio.html#istio_tutorial)
### Overview of IKS cluster
### Show Istio Helm chart on IBM Cloud page
### From CLI, install Istio helm.
### From CLI, install Bookinfo
[Refer to Istio Bookinfo example.](https://istio.io/docs/examples/bookinfo/)
1. Set namespace label to mark as auto inject
    ```
    kubectl label namespace default istio-injection=enabled
    ```
2. Install bookinfo app from Istio github
    ```
    kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.0/samples/bookinfo/platform/kube/bookinfo.yaml
    ```
3. Set ingress gateway
    ```
    kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.0/samples/bookinfo/networking/bookinfo-gateway.yaml
    ```
4. Get bookinfo product page URL
    ```
    # Get host
    export ISTIO_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    # Get port
    export ISTIO_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
    # Echo full URL
    echo http://$ISTIO_HOST:$ISTIO_PORT/productpage
    ```
6. Open bookinfo product page at http://$ISTIO_HOST:$ISTIO_PORT/productpage
7. Apply default destination rules - configure all subsets for the service versions
    ```
    kubectl apply -f destination-rule-all.yaml
    ```

### Run Bookinfo scenario
[Traffic management tutorial](https://istio.io/docs/tasks/traffic-management/request-routing/)
1. **Default routing**
    ```
    kubectl delete -f virtual-service-all-v1.yaml
    ```
    1. Refresh productpage to show random review service
2. **Routing to 1 version only**
    1. Apply `virtual-service-all-v1.yaml` to send all requests to v1
        ```
        kubectl apply -f virtual-service-all-v1.yaml
        ```
    2. Refresh productpage to show only v1 of reviews is called
    3. Show Kiali Traffic
3. **Content based routing**
    1. Apply `virtual-service-reviews-test-v2`
    2. Login to productpage as `jason`. All responses use reviews v2 which calls ratings v1.
    3. Anonymous and any other user routes to `reviews` v1.
4. **Fault Injection**
    1. Apply delay in call to `ratings` for Jason.
        ```
        kubectl apply -f virtual-service-ratings-test-delay.yaml
        ```
        For Jason only, responses from `ratings` is delayed by 7 seconds. This exposes a hardcoded timeout of 6 seconds in `productpage` service (coded are 3 second timeout plus 1 retry).

        Issue is fixed in v3 of `ratings`. So change `virtualservice` rule to route all traffic to v3 for Jason.
    2. Apply HTTP abort failure
        ```
        kubectl apply -f virtual-service-ratings-test-abort.yaml
        ```
5. **Circuit Breaker**
    1. Deploy `httpbin` and set destination rules
        ```
        kubectl apply -f httpbin.yaml
        kubectl apply -f httpbin-destination-rule-CB.yaml
        ```
    2. Deploy test client `fortio` and exec into the container
        ```
        kubectl apply -f fortio-deploy.yaml
        # Export fortio pod so we can use it
        export FORTIO_POD=$(kubectl get pod | awk '/fortio/ {print $1}')
        kubectl exec -it $FORTIO_POD -c fortio /usr/local/bin/fortio -- load -curl  http://httpbin:8000/get
        ```
        Should get a 200 status. So now we are ready to break it.
    3. CB is set for 1 connection and 1 pending request. So we'll swamp the service with 2 clients and send 20 requests.
        ```
        kubectl exec -it $FORTIO_POD -c fortio /usr/local/bin/fortio -- load -c 2 -qps 0 -n 20 -loglevel Warning http://httpbin:8000/get
        ```
    4. Most requests actually succeeded
        ```
        Code 200 : 19 (95.0 %)
        Code 503 : 1 (5.0 %)
        ```
    5. Ramp up the clients to 3
        ```
        kubectl exec -it $FORTIO_POD -c fortio /usr/local/bin/fortio -- load -c 3 -qps 0 -n 20 -loglevel Warning http://httpbin:8000/get
        ```
    6. More requests are failing
        ```
        Code 200 : 12 (60.0 %)
        Code 503 : 8 (40.0 %)
        ```
    7. Investigate istio tracing to see more details
        ```
        kubectl exec -it $FORTIO_POD  -c istio-proxy  -- sh -c 'curl localhost:15000/stats' | grep httpbin | grep pending
        ....
          cluster.outbound|8000||httpbin.default.svc.cluster.local.upstream_rq_pending_overflow: 35
          cluster.outbound|8000||httpbin.default.svc.cluster.local.upstream_rq_pending_total: 47
        ```
        `pending_overflow` count is number of requests that were trapped by circuit breaking.
