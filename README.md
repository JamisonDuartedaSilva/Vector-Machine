# Vector Machine

Fork independente do [VectorCraft](https://github.com/storytold/vectorcraft), criado originalmente pela equipe ArtCraft. Editor vetorial em Rust para desktop e navegador (WebAssembly). Este fork não é um produto oficial da ArtCraft nem da Adobe.

**Estado:** preparação da base, ainda em alpha. Os nomes dos crates, executáveis, preferências e arquivos `.vectorcraft` continuam iguais aos do upstream para preservar compatibilidade. A identidade visual completa do aplicativo ainda está pendente. Não há release próprio publicado.

## Fluxo de desenvolvimento

- `main`: referência inicial herdada do fork; não é nosso destino de PRs.
- `develop`: integração. Criar branches de trabalho e abrir PRs para esta branch.
- `master`: versão aprovada pelo Jamison. Promoção e merge ficam com ele.

O GitHub ainda pode mostrar `main` como branch padrão. Em **Settings → General → Default branch**, selecionar `develop` para sugerir o destino correto dos PRs. Em **Settings → Rules → Rulesets**, proteger `develop` e `master`, exigir PR e os checks da CI; conceder bypass de `master` somente ao responsável por releases. Esses ajustes administrativos não são aplicados pelo workflow.

## Executar localmente

Instalar Rust estável com [rustup](https://rustup.rs/), versão 1.95 ou superior. Em Debian/Ubuntu, instalar as dependências nativas:

```sh
sudo apt-get update
sudo apt-get install -y build-essential pkg-config libxkbcommon-dev libwayland-dev libx11-dev libxrandr-dev libxi-dev libgl1-mesa-dev
git clone https://github.com/JamisonDuartedaSilva/Vector-Machine.git
cd Vector-Machine
git switch develop
rustup target add wasm32-unknown-unknown
cargo run --locked --release -p vectorcraft
```

No Windows, usar Rust MSVC e as ferramentas C++ do Visual Studio; no macOS, instalar as Command Line Tools do Xcode. Detalhes do motor e das plataformas: [docs/development.md](docs/development.md).

### Versão web

```sh
cargo install trunk --version 0.21.14 --locked
cd apps/vectorcraft-web
trunk serve --release
```

Abrir `http://127.0.0.1:8766`. Para gerar os arquivos estáticos, usar `trunk build --release`; a saída fica em `dist/web`. Não há backend obrigatório. Salvar/exportar antes de fechar o editor.

### Qualidade e automação

```sh
cargo xtask ci
cargo build --locked -p vectorcraft
cargo run --locked -p vectorcraft-cli -- run --in examples/dusk-poster.vectorcraft --export /tmp/vector-machine-smoke.png
cargo run --locked --release -p vectorcraft-cli -- mcp
```

`cargo xtask ci` verifica formatação, Clippy, testes, atribuição dos assets, marcas, dependências entre camadas e compilação WASM. A exportação pela CLI valida uma operação real sem abrir janela. O MCP deve receber diretórios permitidos conforme [docs/mcp.md](docs/mcp.md).

## GitHub Actions

[Vector Machine CI](.github/workflows/vector-machine-ci.yml) roda em PRs para `develop`/`master`, pushes nessas branches e execução manual:

1. Um único job compila a versão web com dependências travadas e cache de Rust.
2. Verifica que o HTML e o WASM foram gerados, que o lockfile não mudou e envia `vector-machine-web` como artefato, retido por sete dias.

A CI cotidiana não executa `cargo xtask ci`, Clippy, a suíte inteira de testes ou builds desktop. Esses comandos continuam disponíveis para validação local e mudanças que precisem deles. Alterações apenas em documentação não disparam o build; execução manual permanece disponível. Novas execuções cancelam a anterior da mesma branch/PR. O job tem limite de 30 minutos, não uma estimativa de duração; a primeira compilação sem cache pode levar mais tempo.

O artefato pode ser baixado em **Actions → execução → Artifacts** e servido por um servidor estático. Não há deploy, publicação de releases ou merge automático. A primeira execução precisa ser aprovada no GitHub se os Actions estiverem desativados no fork. Nenhum secret é necessário nesta etapa.

Os workflows herdados continuam no repositório, mas seus jobs estão limitados ao upstream `storytold/vectorcraft`. Isso evita ativar neste fork os agendamentos e instaladores oficiais, inclusive runners maiores. Teremos uma pipeline própria de distribuição quando a identidade do aplicativo estiver pronta.

## Atualizar a base

```sh
git remote add upstream https://github.com/storytold/vectorcraft.git
git fetch upstream
git switch develop
git switch -c chore/update-upstream
git merge upstream/main
cargo xtask ci
git push -u origin chore/update-upstream
```

Abrir PR para `develop` e revisar conflitos, workflows e branding. Se o remote `upstream` já existir, omitir o primeiro comando. Não substituir nossas branches nem fazer force-push.

## Limitações e licença

A compatibilidade com `.ai` ainda é parcial; a presença de uma ferramenta não implica paridade completa com o Illustrator. As avaliações e o roadmap herdados descrevem o upstream, não resultados medidos deste fork: [ROADMAP.md](ROADMAP.md), [docs/gaps.md](docs/gaps.md).

Código sob **MIT ou Apache-2.0**, à escolha. Mantidos [LICENSE-MIT](LICENSE-MIT), [LICENSE-APACHE](LICENSE-APACHE), [NOTICE](NOTICE) e [ASSETS.md](ASSETS.md). As imagens de marca ArtCraft foram removidas; atribuições e licenças de terceiros permanecem. Respeitar as regras de implementação independente e de assets em [AGENTS.md](AGENTS.md).
