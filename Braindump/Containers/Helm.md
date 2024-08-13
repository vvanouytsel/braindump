* Show the values deployed together with a helm chart

```bash
❯ helm get values -n mynamespace mychart
USER-SUPPLIED VALUES:
containerMode:
  type: dind
```

* Search for a chart in all repositories

```bash
❯ helm search repo falcon
NAME                                  	CHART VERSION	APP VERSION	DESCRIPTION                                       
crowdstrike/falcon-image-analyzer     	1.1.8        	1.0.13     	A Helm chart for Falcon Image Analyzer            
crowdstrike/falcon-integration-gateway	0.5.1        	3.2.0      	Falcon Integration Gateway for cloud              
crowdstrike/falcon-kac                	1.1.2        	1.1.2      	A Helm chart to deploy CrowdStrike Falcon Kuber...
crowdstrike/falcon-sensor             	1.29.1       	1.29.1     	A Helm chart to deploy CrowdStrike Falcon senso...
```

* Search for a chart with major version 1 in all repositories

```bash
❯ helm search repo falcon --version ^1  
NAME                             	CHART VERSION	APP VERSION	DESCRIPTION                                       
crowdstrike/falcon-image-analyzer	1.1.8        	1.0.13     	A Helm chart for Falcon Image Analyzer            
crowdstrike/falcon-kac           	1.1.2        	1.1.2      	A Helm chart to deploy CrowdStrike Falcon Kuber...
crowdstrike/falcon-sensor        	1.29.1       	1.29.1     	A Helm chart to deploy CrowdStrike Falcon senso...
```