# Fluxo que roda a partir de qualquer arquivo do tenant

Utilizar o seguinte gatilho, o site e a lista base não importam.
<img src="https://imgur.com/h6J0X3f.png" width="800" height="200">

Aqui só é necessário inicializar as variáveis Contador e NomeBiblioteca e varRootSite, com o valor padrão sendo https://contoso.sharepoint.com/sites/
<img src="https://imgur.com/c515wiD.png" width="800" height="400">

Para a variável itemURL, utilize o valor do próprio trigger: `triggerBody()?['entity']?['itemUrl']` e varSite vazio.
<img src="https://imgur.com/5XOJzYH.png" width="800" height="400">

Nessa print o nome das constantes/variáveis estão errados, o nome ideal seria 1. SetSiteName 2. SetSiteUrl. 

No primeiro compose, utilize a seguinte função para obter apenas o nome do site (não necessário): 
`substring(variables('varItemURL'),length(variables('varSiteRoot')),indexOf(substring(variables('varItemURL'),length(variables('varSiteRoot'))),'/'))`

Defina a variável varSite como: `concat(variables('varSiteRoot'), outputs('SiteNameUrl'))`
<img src="https://imgur.com/QUr3cts.png" width="800" height="300">

Utilize a ação obter todas as listas e bibliotecas do site, e passe como valor a variável varSite, que no caso será a URL do site no qual foi chamado o fluxo.

Após isso, iremos utilizar um aplicar a cada para cada valor da ação anterior, logo no inicio fazemos uma verificação na variável contador (poderia ser um boolean também).

Esta verificação é feita inicialmente para evitar que o aplicar a cada continue fazendo chamadas no sharepoint desnecessariamente, como o Power Automate não possui a instrução ```break``` para sair de um loop.

Se o valor for 0, então devemos fazer a chamada no Sharepoint.

<img src="https://imgur.com/yiSLcMl.png" width="800" height="400">

Utilize o varSite como parâmetro para url base e utilize a seguinte url para query: 
`_api/web/lists/getByTitle('@{items('Aplicar_a_cada_3')?['DisplayName']}')/items?$select=FieldValuesAsText/FileRef&$expand=FieldValuesAsText&$filter=FileLeafRef eq '@{triggerBody()?['entity']?['fileName']}' and ID eq @{triggerBody()?['entity']?['ID']}`

Onde: DisplayName vem da ação Obter todas listas e bibliotecas, fileName vem do triggerBody e ID também.

Utilizo um parse Json no resultado, não é necessário, a estrutura da resposta segue o seguinte esquema:
```
{
    "type": "object",
    "properties": {
        "d": {
            "type": "object",
            "properties": {
                "results": {
                    "type": "array",
                    "items": {
                        "type": "object",
                        "properties": {
                            "__metadata": {
                                "type": "object",
                                "properties": {
                                    "id": {
                                        "type": "string"
                                    },
                                    "uri": {
                                        "type": "string"
                                    },
                                    "etag": {
                                        "type": "string"
                                    },
                                    "type": {
                                        "type": "string"
                                    }
                                }
                            },
                            "FieldValuesAsText": {
                                "type": "object",
                                "properties": {
                                    "__metadata": {
                                        "type": "object",
                                        "properties": {
                                            "id": {
                                                "type": "string"
                                            },
                                            "uri": {
                                                "type": "string"
                                            },
                                            "type": {
                                                "type": "string"
                                            }
                                        }
                                    },
                                    "FileRef": {
                                        "type": "string"
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
```
Logo após verifico se a variável Contador ainda é 0 e se o tamanho do array da propriedade ['d']?['results'] é maior que zero, 
caso seja, definimos o Contador como 1 e a variável NomeBiblioteca como o DisplayName da iteração.

Fora do aplicar a cada, apenas verifico se a variável Contador é igual a 1 e que o nome da biblioteca não seja nulo, (pode ser que não tenha encontrado em nenhum item)

<img src="https://imgur.com/XhbeD1W.png" width="800" height="400">

Com isso, caso seja verdadeiro, apenas passo como parâmetros para a ação Obter propriedades de arquivo, varSite, NomeBiblioteca e o ID (Id do `triggerBody()?['entity']?['ID']`)

<img src="https://imgur.com/uZC9d0u.png" width="800" height="400">

Com isso nós já temos tudo relacionado à aquele arquivo, os metadados da coluna, contexto da biblioteca e do site executado etc. 
