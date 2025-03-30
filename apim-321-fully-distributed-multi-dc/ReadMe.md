This is to try out the TCP connections of TM and GW when there is a Multi DC setup.

1. Steps used to build the image.

   FROM docker.wso2.com/wso2am:3.2.1.38
   COPY wso2carbon.jks wso2am-3.2.1/repository/resources/security/wso2carbon.jks
   COPY client-truststore.jks wso2am-3.2.1/repository/resources/security/client-truststore.jks
   COPY executionplans /home/wso2carbon/wso2-tmp/executionplans
   COPY eventpublishers /home/wso2carbon/wso2-tmp/eventpublishers
   COPY eventreceivers /home/wso2carbon/wso2-tmp/eventreceivers
   COPY eventstreams /home/wso2carbon/wso2-tmp/eventstreams
   COPY eventpublishersTokeRevocationj2/ /home/wso2carbon/wso2-config-volume/repository/resources/conf/templates/repository/deployment/server/eventpublishers/
   COPY jndiPropertiesj2/ /home/wso2carbon/wso2-config-volume/repository/resources/conf/templates/repository/conf/

2. Need to create a docker-registry type secret named 'janithcmw-secret' to access the images that are hosted in the pvt repo.

   docker-registry <secret-name> --docker-username=<username> --docker-password=<dckr_pat_xxxxxxxx> --docker-email=<email>   

3. Install the DC-1 helm chart

   helm install apim-321-multi-dc-aks /home/janith/apim-multi-dc-deployment/apim-321-fully-distributed-multi-dc/dc-1

4. Install the DC-2 helm chart

   helm install apim-321-multi-dc-aks /home/janith/apim-multi-dc-deployment/apim-321-fully-distributed-multi-dc/dc-2

5. Up to now the DNS handling is done manually. So the hostAliases used to manage the DNS entries.

6. To test the events HTTP eventreceivers has been deployed, can be access as follows.
   With in DC
      curl -d '{"event": {"payloadData": {"throttleKey": "/Context/of/no/api", "isThrottled": "true", "expiryTimeStamp": "1726682451000", "evaluatedConditions": "W10="}}}' https://traffic.manager.dc1.am.wso2.com/endpoints/FeedJMSEvent -k
      curl -d '{"event": {"payloadData": {"throttleKey": "/Context/of/no/api", "isThrottled": "true", "expiryTimeStamp": "1726682451000", "evaluatedConditions": "W10="}}}' https://traffic.manager.dc1.am.wso2.com/endpoints/FeedJMSEvent -k
   Across All DCs
      curl -d '{"event": {"payloadData": {"eventId": "revoke-id", "revokedToken": "xxxxxxxxx", "ttl": "ttl", "expiryTime": "123456789", "type": "test-dc1-tm", "tenantId": -1234}}}' https://traffic.manager.dc1.am.wso2.com/endpoints/FeedTokenRevocationEvent -k
      curl -d '{"event": {"payloadData": {"eventId": "revoke-id", "revokedToken": "xxxxxxxxx", "ttl": "ttl", "expiryTime": "123456789", "type": "test-dc2-tm", "tenantId": -1234}}}' https://traffic.manager.dc2.am.wso2.com/endpoints/FeedTokenRevocationEvent -k