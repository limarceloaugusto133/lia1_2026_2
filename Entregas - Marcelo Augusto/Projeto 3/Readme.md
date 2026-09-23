# 🏭 Inspetor Visual de Defeitos em Peças Fundidas

Um modelo de Inteligência Artificial (CNN residual em PyTorch) que detecta defeitos em peças fundidas na linha de produção, acompanhado de um **Inspetor Universal ONNX**: um aplicativo web que roda a inferência direto no navegador, com este modelo ou com qualquer outro modelo `.onnx`.

### 🌐 Acesse o aplicativo: **[inspetor-universal-onnx.vercel.app](https://inspetor-universal-onnx.vercel.app/)**

<img src="Imagens%20da%20Inferencia/Peca_com_defeito.png" width="600" alt="Demonstração do Inspetor para peça com defeito">

<img src="Imagens%20da%20Inferencia/Peca_sem_defeito.png" width="600" alt="Demonstração do Inspetor para peça sem defeito">

## 🎯 Objetivo
Automatizar a inspeção visual de peças fundidas (impulsores de bomba submersa). O modelo classifica as imagens separando as peças defeituosas das aprovadas, com o foco em **minimizar o erro mais caro: deixar passar uma peça com defeito**.

## 🚀 Teste o modelo no navegador
Não é preciso instalar nada nem rodar o Colab. A inferência acontece **100% no seu navegador** (ONNX Runtime Web), então as imagens enviadas não saem do seu computador.

1. Baixe o arquivo `inspetor_fundidos.onnx` da pasta `Modelo Onnx/` deste repositório.
2. Acesse **[inspetor-universal-onnx.vercel.app](https://inspetor-universal-onnx.vercel.app/)**.
3. Clique no botão **.onnx** e selecione o arquivo baixado.
4. Confira o cartão de metadados: classes (`ok` / `defeito`), formato de entrada, normalização e limiar de decisão são lidos automaticamente do próprio `.onnx`.
5. Envie uma foto da peça (vista superior, JPG/PNG) e clique em **Inspecionar**.
6. Veja o veredito e a probabilidade de defeito. Se quiser, ajuste o slider de **confiança mínima**.

> 💡 A primeira inferência pode demorar alguns segundos, porque o navegador baixa o runtime do ONNX uma única vez. As seguintes são quase instantâneas.

## 🧩 O Inspetor Universal ONNX
O aplicativo não tem nenhuma classe, tensor ou arquitetura fixada no código. Ele lê os metadados do modelo enviado e monta a interface de acordo com o que encontra, por isso também funciona com outros modelos:

- **Classificadores** (saída escalar ou `[1, C]`), como o deste projeto.
- **Detectores estilo YOLO** (v5, v7, v8, v11), com caixas desenhadas sobre a imagem.
- **Outros formatos** de saída: o app não trava e mostra as estatísticas brutas dos tensores (shape, mín., máx., média).

Formatos aceitos no upload: `.onnx` sozinho, ou um `.zip` com o `.onnx` e um `config.json` opcional.

### Regras de decisão
Além da confiança mínima, é possível escrever regras próprias no formato `Rótulo | expressão`, usando o nome da classe como variável:

```
Reprovar | defeito >= 0.5
Reinspecionar | defeito >= 0.06 && defeito < 0.5
```

### `config.json` opcional
Útil para modelos que não trazem metadados embutidos (por exemplo, um YOLO exportado sem nomes de classes). Todos os campos são opcionais:

```json
{
  "nome_problema": "Buraco na via",
  "nome_modelo": "YOLO11n",
  "classes": { "0": "buraco" },
  "metricas": { "map50": 0.608, "map50_95": 0.238, "epoca": 36 },
  "entrada": { "formato": "NCHW", "tipo_dado": "RGB", "preprocessamento": "letterbox" },
  "confianca_minima_padrao": 0.35,
  "regras_decisao": "Buraco na via | buraco >= 0.5"
}
```

> ⚠️ O `inspetor_fundidos_config.json` gerado pelo notebook é um registro do treino, não precisa ser enviado ao app. O modelo deste projeto já carrega tudo que o app precisa dentro do próprio `.onnx`.

## ▶️ Como treinar o modelo (Notebook)
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/limarceloaugusto133/lia1_2026_2/blob/main/Visualizar_pe%C3%A7as_com_defeito.ipynb)

Abra o arquivo `Visualizar_peças_com_defeito.ipynb` no Google Colab (de preferência com GPU T4) e rode as células em ordem. Ele faz todo o trabalho pesado:
1. Baixa o dataset do Kaggle automaticamente (`kagglehub`).
2. Faz uma limpeza não destrutiva e uma análise exploratória, checando se a iluminação sozinha "entrega" a classe.
3. Audita vazamento de dados: agrupa quase-duplicatas por hash perceptual (invariante a giros e espelhamentos) e separa treino/validação/teste **por grupo**.
4. Compara com baselines ingênuos (classe majoritária e regressão logística).
5. Treina a CNN residual com AMP, early stopping e agendador de taxa de aprendizado.
6. Avalia no teste uma única vez, com intervalos de confiança por bootstrap, análise de erros e Grad-CAM.
7. Escolhe o limiar de decisão pelo **custo de negócio** (falso negativo vs. falso positivo).
8. Exporta `.pth`, `config.json` e `.onnx`, validando que o ONNX dá o mesmo resultado que o PyTorch.

A seção 16 do notebook ainda permite subir o protótipo antigo pelo Colab com um túnel temporário, mas o jeito recomendado de testar agora é o link da Vercel acima.

## 📊 Resultados (conjunto de teste)
| Métrica | Valor |
|---|---|
| AUC-ROC | 0,9994 |
| Recall "defeito" (limiar 0,06) | 0,9967 |
| Precisão "defeito" | 0,9935 |
| Especificidade | 0,9906 |
| Acurácia | 0,9942 |
| Falsos negativos / positivos | 2 / 4 (em 1.039 imagens) |
| Meta de recall (≥ 0,98) | ✅ Atingida |

Para comparação, a regressão logística chegou a 0,91 de acurácia na validação. O modelo tem cerca de 1,6 milhão de parâmetros, entrada `1 × 128 × 128` em escala de cinza, e treinou em cerca de 3,5 minutos numa T4.

## 📁 Estrutura do Projeto
- `Visualizar_peças_com_defeito.ipynb` → O código completo com as explicações passo a passo.
- `Modelo Onnx/` → Contém o modelo exportado (`.onnx`) e as configurações do treino (`.json`).
- `Arquivo do Lovable/` → Contém o projeto (`.zip`) do protótipo.
- `Imagens da Inferencia/` → Capturas de tela da demonstração.

O código do aplicativo está em um repositório separado, publicado na Vercel com deploy automático a cada alteração.

## ⚠️ Limitações
- O modelo foi treinado para um cenário específico (iluminação, enquadramento e vista superior da câmera). Mudanças nesses fatores reduzem a precisão.
- Ele indica se a peça está "OK" ou "Com Defeito", mas não identifica o tipo (trinca, porosidade, rebarba) nem a localização do defeito.
- Só vale para este impulsor: outra peça, molde ou fornecedor exige novos dados.
- Com uma taxa de defeito realista de produção (ex.: 3%), a precisão cai para cerca de 0,77, mesmo sem mudar o modelo.
- No aplicativo universal, arquiteturas diferentes de classificador e YOLO (segmentação, pose) caem no modo de estatísticas brutas.

## 📚 Créditos e Licença
- **Dataset:** Ravirajsinh Dabhi (Kaggle) com imagens da PILOT TECHNOCAST.
- **Inferência:** ONNX Runtime Web (Microsoft).
- **Interface:** Protótipo gerado com Lovable, publicado na Vercel.