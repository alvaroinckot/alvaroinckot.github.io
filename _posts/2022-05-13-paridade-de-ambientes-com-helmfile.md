---
layout: post
title:  "Paridade de ambientes com Helmfile"
date:   2022-05-13 01:00:00 -0300
categories: [ devsecops, tutorial ]
tags: [pt-BR]
permalink: /posts/paridade-de-ambientes-com-helmfile
---

A partir do momento que começamos a desenvolver sistemas para a nuvem, [reduzir a lacuna de ferramentas entre ambiente de desenvolvimento e produção é sempre um objetivo](https://12factor.net/pt_br/dev-prod-parity) e ao mesmo tempo um grande desafio, pois até mesmo com o advento dos containers e demais soluções de virtualização, algumas arestas ainda ficam pendentes de serem aparadas.

Um caso muito comum em empresas dos mais variados tamanhos, é o Kubernetes em ambiente produção e docker compose em ambiente local. Essa dupla dinâmica resolve 90% dos problemas e fornece uma similaridade suficientemente satisfatória para desenvolvimento de sistemas, aferições de qualidade, e até mesmo testes de integração.

Porém, ainda se tratam de dois orquestradores de containers complemetamente diferentes, com natuzeras e complexidades diametralmente opostas. O Kubernetes oferece inúmeros recursos para a abstração de componentes de infraestrutura, escalonamento, e gerenciamento de workloads, o que muita das vezes o torna muito complexo de ser utilizado em ambientes locais, tornando o docker compose muito mais atrativo, principalmente por sua simplicidade na hora de montar um set de serviços. 

Fica a pergunta, quais são esses casos onde vale a pena usar o Kubernetes local? Existem algumas vantagens em se ter o Kubernetes como first-class citizen na sua arquitetura, pois recursos como CronJobs, PersistentVolumes, Ingresses, não só facilitam o dia-dia do desenvolvedor, como também enriquecem o famoso YBIYRI (You build it, You run it!), afinal são essas abstrações de infraestrutura que permitem nós meros mortais desenvolvedores a manterem uma aplicação de larga escala sem perder a sanidade mental (carece de fontes).

Outro ponto importante é na visão de esforço de desenvolvimento, pois manter um único orquestrador poupa muitas horas de pesquisa e setup em situações onde novas ferramentas são necessárias. Por exemplo, se o time decidir que precisa de uma aplicação para fazer o tracing das chamadas de APIs, ao invés de estudar como aplicá-la em ambos os orquestradores, um simples helm chart pode resolver o problema com Kubernetes.

Existem algumas situações também onde o Kubernetes se tornar overkill para certas aplicações, e o docker compose seria mais adequado para ser usado em produção, mas isso é papo pra outro post.

### Kubernetes + Helm + Helmfile

Mas então, como resolvemos a complexidade do Kubernetes em ambiente local? A resposta começa com [Helm](https://helm.sh) e [Helmfile](https://github.com/helmfile/helmfile). O Helm é um gerenciador de aplicações de Kubernetes, ou nas palavras deles, pense como um apt/yum/homebrew para Kubernetes. Já o Helmfile, é um agrupador de desses pacotes, praticamente um docker-compose para Helm charts.

Um exemplo de docker-compose.yml:

```yaml
version: "3.9"

services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
volumes:
  db_data: {}
```

O mesmo helmfile.yaml:

```yaml
repositories:
  - name: bitnami
    url: https://charts.bitnami.com/bitnami

  - name: wordpress
    chart: bitnami/wordpress
    version: '~14.10.8'
```

> O arquivo de orquestração se torna muito mais simples, pois o próprio chart define COMO o serviço será executado.

O helmfile suporta algumas coisas legais como: interpolação de variaveis, multiplos arquivos, definição de ambientes, injeção de variáveis de ambiente, delta deploy….

### O Kubernetes no ambiente local

Helm e Helmfile apresentados, agora fica outro problema pendente, como executar Kubernetes local de maneira eficiente? Se você usa o Docker for Desktop basta habilitar a opção Kubernetes e *voilà*, cluster configurado e pronto para o uso, mas é claro, as opções não se restringem a essas. A primeira opção, principalmente pra quem usa MacOS, é o [Rancher Desktop](https://rancherdesktop.io/)  que fornece junto consigo uma versão do Kubernetes mais leve, o K3S, que usa menos RAM e CPU mas que quase nada difere no seu funcionamento (eles até sugerem seu uso para aplicações IoT). Outra opção é o [Minikube](https://minikube.sigs.k8s.io/docs/start/) que também é install and play, porém sem a opção de bind automatico de portas no localhost.

Espero que este post seja útil na sua jornada. Nos vemos em breve!

### Links

- [https://12factor.net/pt_br/dev-prod-parity](https://12factor.net/pt_br/dev-prod-parity)
- [https://helm.sh](https://helm.sh)
- [https://github.com/helm/helm](https://github.com/helm/helm)
- [https://github.com/helmfile/helmfile](https://github.com/helmfile/helmfile)
- [https://helmfile.readthedocs.io/en/latest/](https://helmfile.readthedocs.io/en/latest/)
- [https://rancherdesktop.io/](https://rancherdesktop.io/)
- [https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)
- [https://github.com/bitnami/charts/tree/master/bitnami/wordpress](https://github.com/bitnami/charts/tree/master/bitnami/wordpress)