# 🏭 Inspetor Visual de Defeitos em Peças Fundidas

Um modelo de Inteligência Artificial (CNN em PyTorch) que detecta defeitos em peças na linha de produção. A demonstração roda direto no navegador, pela interface do lovable e é possível acessar rodando a secção 16 no colab!

<img src="Entregas - Marcelo Augusto/Projeto 3/Imagens da Inferência/Peça_com_defeito.png" width="600" alt="Demonstração do Inspetor para peça com defeito">

<img src="Entregas - Marcelo Augusto/Projeto 3/Imagens da Inferência/Peça_sem_defeito.png" width="600" alt="Demonstração do Inspetor para peça sem defeito">

## 🎯 Objetivo
Automatizar a inspeção visual de peças fundidas (impulsores de bomba). O modelo classifica as imagens separando as peças defeituosas das aprovadas, com o foco em **minimizar o erro mais caro: deixar passar uma peça com defeito**.

## 🚀 Teste o modelo no navegador
Você pode testar a IA direto pelo navegador. A inferência é feita baixando os arquivos das pastas "Modelo Onnx"e "Arquivo Lovable" e inserindo eles nas células 16.1 e 16.4 respectivamente.

## ▶️ Como treinar o modelo e acessar sua inferência(Notebook)
[![Open In Colab](https://github.com/limarceloaugusto133/lia1_2026_2/blob/main/Visualizar_pe%C3%A7as_com_defeito.ipynb)]

Abra o arquivo `Visualizar_peças_com_defeito.ipynb` no Google Colab e rode as células em ordem. Ele faz todo o trabalho pesado:
1. Baixa o dataset do Kaggle automaticamente.
2. Trata os dados evitando vazamentos de informações (data leakage).
3. Treina a rede neural (CNN).
4. Exporta o modelo final para o formato `.onnx` (que alimenta a demo web).
5. Baixa os artefatos do modelo `.onnx`e `.json` (se os arquivos vieram uma sessão diferente).
6. Baixa o projeto `.zip` do Lovable.
7. Sobe o servidor, cria um túnel e gera a URL da inferência.

## 📁 Estrutura do Projeto
- `Visualizar_peças_com_defeito.ipynb` → O código completo com as explicações passo a passo.
- `Modelo Onnx/` → Contém o modelo exportado (`.onnx`) e as configurações (`.json`).
- `Arquivo do Lovable/` → Contém o projeto (`.zip`) do Lovable.

## ⚠️ Limitações
- O modelo foi treinado para um cenário específico (iluminação e enquadramento exatos da câmera). Mudanças nesses fatores reduzirão a precisão.
- Ele indica se a peça está "OK" ou "Com Defeito", mas não identifica o tipo exato do defeito (trinca, porosidade, etc).

## 📚 Créditos e Licença
- **Dataset:** Ravirajsinh Dabhi (Kaggle) com imagens da PILOT TECHNOCAST.
- **Inferência:** ONNX Runtime Web (Microsoft).
- **Interface:** Protótipo gerado com Lovable.