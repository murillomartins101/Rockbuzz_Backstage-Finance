# 🚨 INSTRUÇÕES PARA CORREÇÃO MANUAL

## Problema
A função `count_shows()` tem código duplicado que causa contagem incorreta (mostra 17 ao invés de 26).

## Linhas a REMOVER no arquivo `rockbuzz_backstage_finance.py`:

### Bloco 1 (aproximadamente linha 612):
```python
qtd_sem_evento += int(desc[com_desc_mask].nunique()) if com_desc_mask.any() else 0
```

### Bloco 2 (aproximadamente linha 626):  
```python
qtd_sem_evento += int(desc[com_desc_mask].nunique()) if com_desc_mask.any() else 0
```

## Como corrigir manualmente:

1. Acesse: https://github.com/murillomartins101/Rockbuzz_Backstage-Finance/edit/main/rockbuzz_backstage_finance.py

2. Procure por (CTRL+F): `qtd_sem_evento += int(desc[com_desc_mask].nunique()) if com_desc_mask.any() else 0`

3. **REMOVA TODAS as ocorrências** desta linha (deve haver 2)

4. **MANTENHA** estas linhas:
```python
qtd_sem_evento += int(desc[com_desc_mask].nunique())
qtd_sem_evento += int(sem_desc_mask.sum())
```

5. Salve o arquivo (Commit changes)

## Verificação:
Após salvar, a contagem deve mostrar **26 shows** ao invés de 17.