📫 📫 📫 
 # Criando templetes .

 ## Vamos descrever de forma rápida e clara de como criar um templete no projeto.
 ## Primeiramente vamos entender a estrutura de arquivos e pastas do projeto:
 
	   /templates ->    Pasta que armazena os templetes criados..
		readme.md -> 
		clienteA.yaml -> Arquivo contendo o "locations"
		
	 	  clienteA -> Pasta do Templete do cliente
			content -> Pasta onde armazena os arquivos do projeto para ser ultilizado no scaffolder. -"skeleton"
			index.yaml -> Arquivo .yaml que descreve o modelo e seus metadados com suas ações que serão executados pelo serviço de scaffolding.
   
 ## Na pasta templates observamos que temos o arquivo clienteA.yaml 
 ## clienteA.yaml -> Esse arquivo armazena as configurações de locations, por exemplo o apontamento para o templete para serviço de scaffolding.
 ## index.yaml -> Contem o metadados, seria os parametros e passos que o serviço de scaffolding vai executar ...


# Exemplo de arquvo de location.

clienteA.yaml

	apiVersion: backstage.io/v1alpha1
	kind: Location                         --> Indica o tipo de arquivo.
	metadata:
	  name: templates-backend
	  description: A collection of backend templates
	spec:
	  targets:
		- ./templates/clienteA/index.yaml  --> aqui está apontando o templete.
		
# Exemplo de arquivo de templete.
## Nesse arquivo pode ter vários passos para determinadas ações, vamos abordar a criação basica de um templete.
 
	apiVersion: scaffolder.backstage.io/v1beta3
	kind: Template -- Indica o tipo de arquivo.
	metadata:
  	name: nodejs-api-template
  	title: Act nodejs Api Template
  	description: Template for the scaffolder that creates a nodejs Api
  	tags: [nodejs, backend]
	spec:
  	owner: user:actdigital
  	type: service
  	-- Esses parâmetros são usados ​​para gerar o formulário de entrada no frontend e são usado para coletar dados de entrada para a execução do scaffolding.
  	parameters:
    	- title: Fill in some steps
      	required:
        	- name
      	properties:
        	name:
          	title: Name
          	type: string
          	description: Unique name of the component
          	ui:autofocus: true
          	ui:options:
            	rows: 5
    	- title: Choose a location
      	required:
        	- repoUrl
      	properties:
        	repoUrl:
          	title: Repository Location
          	type: string
          	ui:field: RepoUrlPicker
          	ui:options:
            	allowedHosts:
              	- dev.azure.com
  	--Essas etapas são executadas no back-end do scaffolder, usando dados que coletamos através dos parâmetros acima.
  	steps:
     	--Cada etapa executa uma ação, neste caso, um arquivo de modelo no diretório de trabalho.
    	- id: fetch-base
      	name: Fetch Base
      	action: fetch:template
      	input:
        	url: ./content
        	values:
          	name: ${{ parameters.name }}
          	project: ${{ (parameters.repoUrl | parseRepoUrl)['owner'] }}
          	repo: ${{ (parameters.repoUrl | parseRepoUrl)['repo'] }}
    	--Esta etapa publica o conteúdo do diretório de trabalho no Azure.
    	- id: publish
      	name: Publish
      	action: publish:azure
      	input:
        	allowedHosts: ['dev.azure.com']
        	description: This is ${{ parameters.name }}
        	repoUrl: ${{ parameters.repoUrl }}
    	--A etapa final é registrar nosso novo componente no catálogo.
    	- id: register
      	name: Register
      	action: catalog:register
      	input:
        	repoContentsUrl: ${{ steps['publish'].output.repoContentsUrl }}
        	catalogInfoPath: '/index.yaml' 
  	--As saídas são exibidas ao usuário após uma execução bem-sucedida do modelo.
  	output:
    	links:
      	- title: Repository
        	url: ${{ steps['publish'].output.remoteUrl }}
      	- title: Open in catalog
        	icon: catalog
        	entityRef: ${{ steps['register'].output.entityRef }}
		 
## Agora que entendemos a estrutura basica de arquivos e parametros, vamos entender como deixar um templete visivel no projeto.

##No seu ambiente de desenvolvimento, em seu projeto procure o arquivo app-config.local.yaml,
##nele vamos adicionar e deixar visivel o templete criado como exemplo a estrutura apresentada anteriormente.
##No arquivo app-config.local.yaml possuem vário parametros, procure o parametro "catalog:" abaixo dele incluia esses parametos.

	catalog:
 	locations:
  	- type: file
   	 target: ../../templates/clienteA.yaml
   	 rules:
     	 - allow: [Template, Location] 
         
##Assim ele ficará visivel para o Catalogo..

- 👀 Dependo do projeto os momes dos Templetes,Location podem mudar.

#Para saber mais sobre criação de templetes consulte: https://backstage.io/docs/features/software-templates/writing-templates/ 



