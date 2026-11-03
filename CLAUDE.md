# IrisDeck — working notes for Claude

## LEIA PRIMEIRO: o IrisDeck é uma cópia modificada do DeckCraft

**Esta seção vale mais que todo o resto do arquivo (e que o AGENTS.md).** O resto veio do DeckCraft; onde ele contradizer esta seção, esta seção vence.

O IrisDeck é uma cópia (fork) do [DeckCraft](https://github.com/storytold/deckcraft), da equipe ArtCraft, adaptada para o ecossistema Iris por **Gabriel Alves Tech**. O repositório é `GabrielAlvesB/IrisDeck` (remote `origin`); o DeckCraft original é o remote `upstream`, usado **só para trazer atualizações**: nunca envie nada para ele. O IrisWord (`../IrisWord`) e o IrisGrid (`../IrisGrid`) são as cópias irmãs do WordCraft e do GridCraft e seguem as mesmas regras.

Todos os ajustes listados abaixo foram feitos de propósito. **Nenhuma atualização vinda do DeckCraft pode desfazê-los.** Ao trazer uma versão nova, o trabalho é incorporar as melhorias do DeckCraft **mantendo o IrisDeck como ele é**. Em qualquer conflito entre o código do DeckCraft e um ajuste do IrisDeck, fica o do IrisDeck. Na dúvida, pergunte ao dono antes de mudar.

### O que é do IrisDeck e não pode ser revertido

1. **Nome visível "IrisDeck"** em tudo que o usuário vê: título da janela, Sobre, Arquivo › Sobre, mensagens, o tema padrão "IrisDeck", PDF e `.pptx` gerados (`Application`, `creator`/`producer`), README, instaladores, pacotes. "DeckCraft" só aparece nos créditos e avisos de copyright (`LICENSE-MIT`, `NOTICE`, as linhas "Based on DeckCraft" e de copyright em `crates/ui-egui/src/dialogs/about.rs` e `crates/ui-egui/src/backstage/info.rs`, a seção de licença do README, `LegalCopyright` em `apps/deckcraft/build.rs` e as frases "DeckCraft contributors" / "DeckCraft authors"); esses avisos são exigidos pela licença e também não podem ser removidos. As fontes de teste "DeckCraft Legacy" e "DeckCraft Colr Squares" mantêm o nome porque ele está gravado dentro dos `.ttf` de teste.
2. **ID do app `com.iris.irisdeck`** (nunca `ai.storyteller.deckcraft`): `packaging/linux/com.iris.irisdeck.*`, `packaging/linux/flatpak/com.iris.irisdeck*.yml`, `packaging/macos/Info.plist.in`, os ícones em `assets/app-icon/hicolor/*/apps/`, `omastore.toml`.
3. **Ícone do IrisDeck** (slide num cavalete com título, duas linhas de texto e gráfico de pizza, gradiente `#ee5f00` → `#b07400`, o mesmo desenho do Workspace do Iris, matiz 48): a fonte é `assets/app-icon/deckcraft.svg`; todo PNG, `.ico` e `.icns` de `assets/app-icon/` é gerado dela. O logotipo do Sobre e da tela Arquivo é o mesmo desenho, feito em código em `paint_logo` (`crates/ui-egui/src/dialogs/about.rs`). Nunca aceite ícones do DeckCraft (nem `icon-source.deckcraft`) por cima destes.
4. **Sem marca ArtCraft**: sem `docs/brand/`, sem logos ArtCraft, sem links para `getartcraft.com` ou `discord.gg/artcraft`, sem itens de Discord (removidos do Sobre, da tela Arquivo, do CLI `links`, do MCP e de `app.links`), sem a lista "Crafting Apps" no README. `crates/engine/src/links.rs` só tem os links do IrisDeck e `UPSTREAM` (crédito ao original). Se uma atualização trouxer menções novas à ArtCraft ou à suíte Crafting Apps na interface, retire-as. A licença de marca da ArtCraft exige isso de forks.
5. **Atualizador** (`UPDATE_APP` em `crates/ui-egui/src/lib.rs`): `name: "IrisDeck"`, `repo: "GabrielAlvesB/IrisDeck"` (exatamente assim: o atualizador compara os links diferenciando maiúsculas), `developer: "Gabriel Alves Tech"`, `accent` `#d45a00`. Nunca volte para `craft_update::suite::DECKCRAFT` puro: o app passaria a baixar o DeckCraft por cima do IrisDeck.
6. **Instalador do Windows** (`packaging/windows/`): `UpgradeCode` `710942A5-BF4E-40C3-B7A3-58FEEEB8343E` (em `deckcraft.wxs` e `install-policy.json`) e GUID do componente `DeckcraftApp` `E484262F-3393-4499-8C9D-344DD5BB0E98` (nunca os do DeckCraft, senão um instalador substitui o outro), fabricante `Gabriel Alves Tech`, pasta `Programs\Iris\IrisDeck`, chaves de registro `Software\IrisDeck`, links de ajuda para `GabrielAlvesB/IrisDeck`. Em `apps/deckcraft/build.rs`, `CompanyName` é `Gabriel Alves Tech`.
7. **Pastas de dados próprias**: no Windows `%APPDATA%\IrisDeck` e no macOS `Application Support/IrisDeck` (preferências, recuperação, dicionários), para não misturar com um DeckCraft instalado. No Linux a pasta continua `~/.config/deckcraft` (nome interno).
8. **CI próprio**: `.github/workflows/ci.yml` é autocontido (sem segredos, token só de leitura, actions fixadas por SHA). **Nunca restaure** os workflows do DeckCraft (`storytold/craft-common-actions@...`, SonarCloud, release, FreeBSD, nix, packaging-lint, windows-arm64), nem `sonar-project.properties` ou `.coderabbit.yaml`: eles rodam código de outra organização e repassam segredos. O Dependabot roda semanalmente (`America/Sao_Paulo`). As instruções de CI mais abaixo neste arquivo são do DeckCraft e **não valem** aqui.
9. **Segurança**: `zip` na versão 9 ou mais nova (`Cargo.toml` do workspace). Não volte para versões antigas (RUSTSEC-2026-0367/0368/0369/0429). Na versão 9, `ZipFile::name()` devolve `Result`: uma entrada com nome ilegível é pulada (`crates/pptx/src/opc.rs`).
10. **Nomes internos mantidos de propósito**: crates `deckcraft-*`, binários `deckcraft.exe` e `deckcraft-cli`, pastas `apps/deckcraft*`, arquivos `assets/app-icon/deckcraft.*`, `packaging/windows/deckcraft.wxs`, a extensão `.deckcraft`, variáveis `DECKCRAFT_*` e os nomes dos arquivos de release (`deckcraft-<versão>-windows-<arq>.msi`, `...-portable.zip`). O atualizador e o Workspace do Iris (`../Iris/src/shared/types/workspace.types.ts`) procuram exatamente esses nomes. Não renomeie.

### Como trazer uma atualização do DeckCraft

1. `git fetch upstream --tags` e escolha a tag nova (por exemplo `v0.16.0`). Faça o merge numa branch: `git switch -c update-deckcraft-0.16.0 && git merge v0.16.0`.
2. Resolva os conflitos mantendo **tudo** da lista acima. Arquivos removidos no IrisDeck (`docs/brand/`, workflows antigos, `sonar-project.properties`, `.coderabbit.yaml`, `assets/app-icon/icon-source.deckcraft`) continuam removidos.
3. Procure o que entrou de novo com a marca antiga e adapte: `git grep -n -e DeckCraft -e ArtCraft -e getartcraft -e discord -e ai.storyteller -e "storytold/deckcraft" -e "Learning Machines"`. Texto visível novo vira IrisDeck; avisos de copyright, créditos e as fontes de teste ficam como estão.
4. Arquivos novos em `.github/workflows/` vindos do DeckCraft: apague. Ícones ou imagens novas de marca: não adote.
5. Se a revisão de `craft-libs` mudou, confira se `craft_update::app::App` ainda aceita os campos do `UPDATE_APP`.
6. Rode `cargo audit`, `cargo xtask assets` e `cargo test --workspace --no-fail-fast`. Nesta máquina, use `CARGO_BUILD_JOBS=4` (com o paralelismo padrão o Windows fica sem memória virtual, os error 1455).
7. **Versão**: a versão do IrisDeck (`[workspace.package]` no `Cargo.toml`) precisa ser **maior que a da última release do IrisDeck**, senão o atualizador não oferece a nova. Rode `cargo update -w` depois de mudar.
8. Mostre ao dono o que mudou antes de fazer merge na `main` e de publicar release.

### Como publicar uma release

1. Versão nova no `Cargo.toml`, `cargo update -w`, commit e push na `main`.
2. Rode `packaging/windows/package.ps1 -Arch x64` num processo `powershell.exe` separado, com a saída indo para arquivo (no PowerShell 5.1, redirecionar dentro do mesmo processo transforma avisos do cargo em erro fatal). Precisa do WiX 5 (`dotnet tool install --global wix --version 5.0.2`). Saída em `dist/release/`.
3. Gere `dist/release/SHA256SUMS.txt` no formato `hash  nome` (dois espaços, sem `*`). O Workspace do Iris não instala sem ele.
4. `gh release create v<versão> -R GabrielAlvesB/IrisDeck --title "IrisDeck v<versão>" --notes-file <notas.md> --latest` com o `.msi`, o `-portable.zip` e o `SHA256SUMS.txt`. Notas em português.
5. Os builds não têm assinatura Authenticode, então o próprio IrisDeck só avisa da versão nova e abre a página da release; a instalação automática acontece pelo Workspace do Iris.

Read [AGENTS.md](AGENTS.md) first: its rules (asset policy, clean room, never crash) bind you and
win over anything here.

New workspace crates must include `[lints] workspace = true` in `Cargo.toml` to inherit
`[workspace.lints]`. Production crate roots must declare the required crate-level lint attributes
before modules or other crate items; crate-level `//!` documentation may precede them.

## Start every session
1. Read `plan/STATUS.md`, then the current milestone in `plan/execution-plan.md`.
2. Run `cargo xtask ci` before you change anything, so you know the tree is green.

## Habits
- Every user-visible action is a command with tests (`crates/engine/src/cmd/*`).
- Look at UI changes: `deckcraft --sample --control 7990` + `ui.screenshot`, or
  `deckcraft-cli render`.
- Commit after each landed arc (`M<n>.<k>: what`) and push to `origin main`.
- Keep `ROADMAP.md`, the progress docs in `docs/` (AGENTS.md §7) and `plan/STATUS.md` current.
- Don't ask the user questions; record genuinely-theirs decisions in `plan/STATUS.md`.

## CI workflows
- Workflows in `.github/workflows/` call the shared workflows in [`storytold/craft-common-actions`](https://github.com/storytold/craft-common-actions) at `@main`, with their inputs and secrets. No other jobs: if no shared workflow does what's needed, it doesn't go here.
- Don't add or change CI in a feature or fix PR. If the existing workflows can't build or test a change, link an existing craft-common-actions issue or PR, or open a new issue, or a PR only for a small fix that is safe for every app and does not increase CI time.
- Some workflow files here still run their own jobs. Don't add to them. Moving a job into craft-common-actions is its own task, never part of another PR, and the job stays here until a shared workflow runs it.
