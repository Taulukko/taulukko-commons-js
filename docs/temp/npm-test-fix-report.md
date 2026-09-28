# npm run test — diagnóstico e correção (parcial)

## Causa raiz
O `npm run test` falhava com:

```
Validation Error:
Preset ts-jest not found relative to rootDir
```

Causa: alterações não commitadas no `package.json` (working tree vs `HEAD`):
- `ts-jest` e `tslint` removidos das `devDependencies`
- `jest` movido de `devDependencies` (`^30.4.2`) para `dependencies` (`^30.5.2`)
- `typescript` movido de `devDependencies` (`^6.0.3`) para `dependencies` (`^7.0.2`)
  - TS 7 é incompatível com o peer do `ts-jest@29.4.x` (`typescript >=4.3 <7`)
- `jasmine-ts` adicionado às `dependencies`, sem nenhum uso em `src/` (grep não retorna nada)
- `jest.config.js` continua exigindo `preset: "ts-jest"`, e `node_modules/ts-jest` não existia

Ou seja: não foi o update do `yargs-parser` que quebrou o teste — foi a reorganização
posterior do `package.json` que removeu o preset que o Jest exige.

## Correção aplicada
Revertido `package.json` + `package-lock.json` para o estado conhecido-bom do `HEAD`:

```
git checkout HEAD -- package.json package-lock.json
```

Isso restaura:
- `devDependencies`: jest `^30.4.2`, ts-jest `^29.4.11`, tslint `^5.11.0`, typescript `^6.0.3`
- `dependencies`: apenas node-cache + yargs-parser (remove jasmine-ts não usado)

## Parte 2 — vulnerabilidades (pedido do Gand, mesma sessão)
Levantamento: `npm audit` = **0 vulnerabilidades**; `audit fix --dry-run` = no-op.
O que assusta no output são avisos de *deprecação* do npm (glob antigo "contém
vulnerabilidades publicizadas"), não advisories. Cadeias encontradas via `npm ls glob`:
- glob 11 ← rimraf 6.0.1 → **corrigido** com `npm update rimraf` (6.1.3, glob 13.0.6).
  `package.json` intacto (caret já cobria); só o lock mudou. Build + testes revalidados.
- glob 10 ← o próprio jest 30.4.2 (upstream; só o Jest pode atualizar). Intocável sem
  risco de partir o runner. NÃO é vulnerabilidade de audit (0 findings).
- glob 7 ← tslint 5.20.1 (devDep direto, linter em uso: `npm run lint` roda e aponta
  erros reais) + test-exclude via ts-jest/babel (intocável). Remover o tslint = decisão
  do Gand (mata script lint + tslint.json + peer warnings do tslint-eslint-rules).
- form-data: 0 ocorrências na árvore (alerta da TASKS.md não se reproduz neste estado).
