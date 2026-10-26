<!-- Versão em português -->
> 🌐 *For the English version of this documentation, please see [README.en.md](README.en.md).*
<br/>

<h1 align="center"> UNANav - Smart Campus Navigation </h1>

<p align="center">
  <img src="files/unanav_logo_principal.png" alt="UNANav Logo"/>
</p>

<br/>
<h2> ℹ️ SOBRE O PROJETO </h2>
<p>
A proposta do projeto UNANav é oferecer um aplicativo que seja capaz de ajudar o usuário a percorrer todo o campus da Universidade de Ribeirão Preto (UNAERP) de forma prática e eficiente. O aplicativo permitirá que o usuário interaja com uma IA usando linguagem natural para chegar ao seu destino, ou através de uma imagem que faz referência ao local destinado, além de poder escolher manualmente para onde deseja ir se quiser (escrevendo o ponto de partida e o seu destino, assim como no Waze, Uber, Google Maps, por exemplo). Se a IA conseguir identificar onde o usuário está através do GPS, ela perguntará somente para onde ele deseja ir dentro do campus (linguagem natural ou imagem que faz referência ao local destino) e o informará sobre seu ponto de partida, caso contrário, ela o informará que não conseguiu localizá-lo dentro do campus e que precisa ser informada sobre o ponto de partida e de destino. A tela de boas vindas permitirá que o usuário possa escolher 1 dos 4 perfis disponíveis (aluno, docente, colaborador e visitante) ou seguir sem perfil se não tiver interesse (servirá somente para algumas sugestões de navegação no campus). A ideia é que o aplicativo possa servir para qualquer pessoa e possa ser usado rapidamente, então, não possuirá sistema de login. Se o usuário escolher um dos 4 perfis disponíveis, as sugestões de navegação farão mais sentido de acordo com o perfil escolhido.
</p>

<h2> 💡 DIFERENCIAIS TÉCNICOS DO UNANav </h2>

* **Busca Inteligente Multimodal (CLIP + pgvector):** Unificação da busca por texto e imagem dentro de um mesmo espaço vetorial ($512$ dimensões). Seja digitando *"biblioteca"* ou enviando uma foto da fachada, o sistema converte a entrada em embeddings e realiza a consulta por similaridade de cosseno diretamente no banco de dados.
* **Tratamento da Degradação de Sinal Indoor (Geofencing + GPS):** Como o GPS residencial/mobile perde precisão dentro de edifícios, o sistema utiliza *Geofencing* (teste ponto-em-polígono no PostGIS) e métricas de *accuracy* da Geolocation API. Ao detectar que o usuário está no interior de um bloco, o app solicita o ponto de partida manual ou via foto do ambiente, garantindo a rota de "última milha" textual.
* **Mapeamento Próprio e Grafo de Caminhos da UNAERP:** Levando em conta que dados públicos (como OpenStreetMap e Google Maps) estavam desatualizados, realizou-se o mapeamento presencial de todo o campus, resultando em 194 feições (63 POIs de banheiros/bebedouros com andar/descrição, 36 polígonos de prédios/quadras e 88 trechos de caminhos perfazendo ~4,5 km de malha conectada).
* **Arquitetura Local-First e Privacidade:** Não há necessidade de conta ou login. Informações pessoais e preferências do usuário (perfil selecionado e histórico recente de navegação) residem exclusivamente no aparelho via banco de dados embarcado (SQLite).

<h2> 🛠️ STACK TECNOLÓGICA </h2>

| Camada | Tecnologia | Função no Projeto |
|---|---|---|
| **Mobile App** | React Native + Expo | Interface cross-platform, captura de fotos e integração nativa. |
| **GPS & Mapas Mobile** | `expo-location` + `react-native-maps` | Leitura de coordenadas GPS e renderização do mapa interativo via tiles abertos do OpenStreetMap (`UrlTile`). |
| **Persistência Local** | SQLite (`expo-sqlite`) | Armazenamento de perfil (Aluno, Docente, Colaborador, Visitante) e histórico local ("Recentes"). |
| **Back-End API** | Python + FastAPI | API RESTful responsável pelos endpoints de geofencing, busca por proximidade e cálculo de rotas. |
| **Banco de Dados** | PostgreSQL 16 | Fonte única de verdade para os dados do campus, mapas e vetores. |
| **Extensão Espacial** | PostGIS 3.6.4 | Consultas geoespaciais avançadas (`ST_DWithin`, `ST_Contains`, `ST_Distance`). |
| **Busca Vetorial** | pgvector 0.8.6 | Armazenamento e consulta de embeddings de texto e imagem gerados pelo modelo CLIP. |
| **IA & Embeddings** | CLIP (OpenAI / OpenCLIP) | Modelo pré-treinado multimodal para conversão de texto/foto em vetores. |
| **Infraestrutura / Cloud** | Oracle Cloud (OCI) + Docker Compose | Servidor VM Ubuntu 22.04 (ARM64) hospedando o PostgreSQL/PostGIS e a API em containers isolados. |

<h2> 🗄️ MODELAGEM DO BANCO DE DADOS (DB_UNANav) </h2>

O banco relacional geoespacial foi estruturado sobre as seguintes entidades principais no PostgreSQL:

<img src="docs/img/DER-UNANav-Project.png"/>

---

* **`tb_predio`**: Entidade lógica dos blocos (ex: Bloco H, Bloco D, Academia).
* **`tb_edificacao`**: Polígonos físicos de cada construção ou quadra/piscina no mapa (`GEOMETRY(Polygon, 4326)`).
* **`tb_poi`**: Pontos de interesse específicos como banheiros e bebedouros (`GEOMETRY(Point, 4326)`) com campo vetorial `embedding VECTOR(512)`.
* **`tb_caminho`**: Malha percorrível do campus composta por calçadas externas e passagens internas cobertas (`GEOMETRY(LineString, 4326)`).

<!--

<h2> 📁 ESTRUTURA DO REPOSITÓRIO </h2>

```text
Projeto-UNANav/
├── .venv/                         # Ambiente virtual Python
├── files/                         # Imagens, diagramas e assets do projeto
│   └── unanav_logo_principal.png
├── src/                           # Código-fonte principal do projeto
│   ├── assets/                    # Recursos visuais para o app mobile
│   ├── .env                       # Variáveis de ambiente (ignorado no Git)
│   ├── schema_unanav.sql          # DDL de criação do banco PostgreSQL/PostGIS
│   ├── importar_dados_db_unanav.py# Script de carga do GeoJSON para o banco
│   ├── testes-de-consultas-de-rotas.sql # Consultas SQL validadas de geofencing e rota
│   ├── main.py                    # Ponto de entrada da API FastAPI (Back-End)
│   └── UNAERP-mapeamento-completo.geojson # Mapeamento geoespacial bruto do campus
├── docker-compose.yml             # Orquestração do PostgreSQL + PostGIS/pgvector na OCI
├── Dockerfile                     # Imagem customizada PostgreSQL 16 para arquitetura ARM64
├── requirements.txt               # Dependências Python do projeto
├── README.en.md                   # Documentação em inglês do repositório
└── README.md                      # Documentação em português do repositório
```

<h2> 🚀 COMO EXECUTAR O PROJETO </h2>

<h3> 1. Pré-requisitos </h3>

* Python 3.10+ instalado.
* Docker & Docker Compose (para execução do banco de dados).
* Node.js & Expo CLI (para execução do aplicativo móvel).

<h3> 2. Configuração do Banco de Dados (Docker / OCI) </h3>

Na pasta raiz onde se encontra o docker-compose.yml:

```text
Bash

# Subir o container do banco de dados PostgreSQL com PostGIS e pgvector
> docker compose up -d --build

# Conectar e aplicar a estrutura física de tabelas
> psql -h localhost -p 5432 -U unanav -d unanav -f src/schema_unanav.sql
```
<h3> 3. Carga Inicial dos Dados Geoespaciais </h3>

No diretório src/, com o ambiente virtual Python ativo:

```text
Bash

# Ativar ambiente virtual (Windows PowerShell)
> .\.venv\Scripts\activate

# Instalar dependências do projeto
> pip install -r requirements.txt

# Executar script de população do banco a partir do GeoJSON
> python src/importar_dados_db_unanav.py
```

<h3> 4. Execução do Back-End (FastAPI) </h3>

```text
Bash

# Iniciar a API em modo de desenvolvimento
> uvicorn src.main:app --reload --host 0.0.0.0 --port 8000
```

> Documentação interativa da API estará disponível em http://localhost:8000/docs.

-->
