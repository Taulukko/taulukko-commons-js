# npm run test — parte 3: remoção do tslint (decisão do Gand)

## Estado
- `package.json`: scripts `lint` e `tsfm` removidos; devDeps `tslint` e
  `tslint-eslint-rules-recommended` removidos. `ts-jest` PRESERVADO
  (numa edição intermediária ele saiu por engano e foi recolocado no passo
  seguinte, antes de qualquer install — sem efeito).
- `tslint.json`: remoção solicitada (`rm`), execução incerta — o shell travou
  (2º hang da sessão) e o watchdog interrompeu. Verificar existência.
- `node_modules`: ainda contém tslint; `npm install` (prune) pendente.
- Sem CHANGELOG.md no repo — a remoção do linter fica registrada aqui e no git.

## Resultado final
- `npm install` podou 32 pacotes (árvore do tslint fora).
- `npm run test`: 6 suites, 65/65 verdes. `npm run build`: OK.
- `npm audit`: 0 vulnerabilidades. `npm audit fix --dry-run`: **0 warnings** —
  o output que motivou a queixa está limpo (saíram os peer warnings do
  tslint-eslint-rules e a cadeia glob 7 do tslint).
- Restam só globs upstream intocáveis (jest→glob 10, ts-jest/babel→glob 7).
- Git: `package.json` modificado, `tslint.json` deletado, `package-lock.json`
  modificado (rimraf 6.1.3 + prune). Sem CHANGELOG.md no repo (não criado;
  registro fica aqui + git).
