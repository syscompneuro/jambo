# Cluster HPC — LFTC / Departamento de Física / UFPE

Guia de início rápido para novos usuários.

Suporte técnico: Paulo Fragoso (NLINK).

### Financiamento

O cluster foi adquirido e é mantido com recursos de projetos coordenados pelo **Prof. Mauro Copelli**:

- CNPq nº 444500/2024-3
- <!-- TODO: completar com os demais financiamentos — agência, número do processo/edital e ano. Mauro pode fornecer a lista. -->
  _(demais projetos — a completar)_

---

## 1. Visão geral

O cluster é composto por um **nó de acesso** (login) e nós de computação gerenciados pelo **Slurm** (gerenciador de filas). Você se conecta ao nó de acesso, prepara seus códigos e dados, e submete *jobs* (o Slurm decide em qual nó de computação eles executam).

| Partição | Nós | CPUs por nó | RAM por nó | GPU | Tempo máximo |
|----------|-----|-------------|------------|-----|--------------|
| `normal` (padrão) | 9 (c1–c7, c9, c10) | 24 | 31 GB | não | 72 h |
| `gpu` | 1 (c11) | 20 | 62 GB | 1× NVIDIA RTX 3060 | 72 h |

- Total da partição `normal`: **216 CPUs**.
- Cada nó da partição `normal` possui disco local `/scratch` (~925 GB) para arquivos temporários de jobs.
- Na partição `normal`, os jobs recebem o **nó inteiro** (alocação exclusiva).


**IMPORTANTE: o nó de acesso serve apenas para editar arquivos, compilar e submeter jobs. Nunca execute processamento pesado nele; use `srun`/`sbatch` para que o trabalho rode nos nós de computação.**

---

## 2. Acesso

### 2.1 Pré-requisitos

1. **Conta no cluster**: solicite ao suporte enviando seu **nome completo** e sua **chave SSH pública** (veja abaixo). Não há acesso por senha, salvo no caso de acesso presencial usando o console.
2. **VPN da UFPE**: para acessar de fora da rede da UFPE, conecte-se primeiro à VPN institucional ([instruções da STI](https://www.ufpe.br/sti)).

### 2.2 Gerando sua chave SSH

**Antes de gerar uma chave nova, verifique se você já tem uma.** Sobrescrever uma chave existente é **irreversível**: se ela for usada em outros sistemas (GitHub, outros servidores), você perde o acesso a eles.

```bash
ls ~/.ssh/
```

- **Já existem `id_ed25519` e `id_ed25519.pub`?** Você já tem uma chave, **não precisa criar outra.** Pule direto para o envio da chave pública (`cat ~/.ssh/id_ed25519.pub`, abaixo).
- **Prefere uma chave exclusiva para o cluster?** Gere com outro nome, sem tocar na existente:

  ```bash
  ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_lftc -C "seu.email@ufpe.br"
  ```

  Nesse caso, use `~/.ssh/id_ed25519_lftc` no campo `IdentityFile` do atalho da seção 2.3.
- **O diretório está vazio ou não existe?** Siga a geração normal abaixo.

Atenção: se durante o `ssh-keygen` aparecer a pergunta `Overwrite (y/n)?`, é porque **já existe uma chave nesse caminho**; responda **`n`** e use uma das opções acima.

No seu computador (Linux, macOS ou Windows PowerShell):

```bash
ssh-keygen -t ed25519 -C "seu.email@ufpe.br"
```

Aceite o local padrão e defina uma senha (passphrase). Isso cria dois arquivos:

- `~/.ssh/id_ed25519`: sua chave **privada** (nunca compartilhe, nem com o suporte);
- `~/.ssh/id_ed25519.pub`: sua chave **pública** (esta você envia ao suporte).

Para ver o conteúdo da chave pública:

```bash
cat ~/.ssh/id_ed25519.pub
```

Envie ao suporte **a linha inteira** (começa com `ssh-ed25519 ...`). O *fingerprint* mostrado na geração não serve para cadastro; precisamos da chave pública completa.

### 2.3 Conectando

Com a conta criada e (se estiver fora da UFPE) a VPN ativa:

```bash
ssh seu.usuario@150.161.8.51
```

Dica: crie um atalho no arquivo `~/.ssh/config` do seu computador:

```
Host lftc
    Hostname 150.161.8.51
    User seu.usuario
    IdentityFile ~/.ssh/id_ed25519
```

Depois disso, basta:

```bash
ssh lftc
```

---

## 3. Transferindo arquivos (upload/download)

Seu diretório `/home/seu.usuario` é **compartilhado entre todos os nós**; o que você envia para o nó de acesso fica visível nos nós de computação.

### Do seu computador para o cluster

```bash
# Um arquivo:
scp dados.csv seu.usuario@150.161.8.51:~/

# Um diretório inteiro:
scp -r meu_projeto/ seu.usuario@150.161.8.51:~/

# Com o atalho "lftc" configurado:
scp -r meu_projeto/ lftc:~/
```

### Do cluster para o seu computador

```bash
scp lftc:~/resultados/saida.dat .
```

### Para volumes grandes ou sincronização (recomendado)

`rsync` retoma transferências interrompidas e envia apenas o que mudou:

```bash
rsync -avz --progress meu_projeto/ lftc:~/meu_projeto/
```

### 3.1 Interface gráfica (arrastar e soltar)

Se preferir não usar a linha de comando, dá para montar seu diretório do cluster como se fosse uma pasta local. Em todos os casos, a VPN da UFPE precisa estar ativa se você estiver fora da rede.

#### Linux (GNOME Files / Nautilus)

1. Abra o **Files** e pressione **Ctrl + L** para abrir a barra de localização.
2. Digite o endereço, trocando pelo seu usuário:

   ```
   sftp://150.161.8.51/home/seu.usuario/
   ```

3. Pressione **Enter**. O Files pedirá autenticação; após autenticar, o diretório `/home/seu.usuario/` aparece como uma pasta comum e você pode **arrastar e soltar** arquivos para fazer upload e download.

O mesmo endereço funciona no Dolphin (KDE) e no Thunar (XFCE), que também aceitam `sftp://`.

#### Windows (FileZilla ou WinSCP) — requer converter a chave

O FileZilla e o WinSCP não leem diretamente a chave no formato OpenSSH: eles usam o formato PuTTY (`.ppk`).

> **Atenção:** o que se converte é a chave **privada** (`id_ed25519`, sem extensão). A chave **pública** (`id_ed25519.pub`) continua sendo enviada ao suporte no formato original — ela não é convertida nem usada aqui.

**Opção A — deixar o FileZilla converter (mais simples)**

1. Menu **Editar → Configurações → Conexão → SFTP**.
2. Clique em **Adicionar arquivo de chave...** e selecione sua chave privada `id_ed25519` (mude o filtro para "Todos os arquivos", já que ela não tem extensão).
3. O FileZilla avisa que a chave não está no formato suportado e pergunta se deseja convertê-la. Clique em **Sim**, informe a passphrase e salve como `id_ed25519.ppk`.
4. Conecte com: Host `sftp://150.161.8.51`, Porta `22`, Usuário `seu.usuario`, deixando a senha em branco (a autenticação usa a chave).

**Opção B — converter manualmente com o PuTTYgen**

1. Instale o [PuTTY](https://www.putty.org/) (o PuTTYgen vem junto) e abra o **PuTTYgen**.
2. Clique em **Load**, mude o filtro para "All Files (*.*)" e selecione `id_ed25519`.
3. Digite a passphrase quando solicitada.
4. Clique em **Save private key** e salve como `id_ed25519.ppk`.
5. No **WinSCP**: novo site → protocolo **SFTP**, host `150.161.8.51`, usuário `seu.usuario` → **Avançado → SSH → Autenticação → Arquivo de chave privada** → aponte para o `.ppk`.

Depois de conectado, tanto o FileZilla quanto o WinSCP mostram dois painéis (local e remoto) e permitem arrastar e soltar.

#### macOS

O Finder não abre `sftp://` nativamente. Use o [Cyberduck](https://cyberduck.io/) (gratuito): **Abrir conexão → SFTP**, servidor `150.161.8.51`, usuário `seu.usuario`, e aponte a chave privada em "Chave SSH privada" — sem necessidade de conversão.

---

## 4. Consultando as especificações do cluster

```bash
# Estado geral: partições, nós e disponibilidade
sinfo

# Detalhe por nó: CPUs, topologia (S:C:T), memória e estado
sinfo -N -l

# Tudo sobre um nó específico:
scontrol show node c11

# Fila de jobs (o que está rodando e aguardando):
squeue

# GPU (ver a placa e sua utilização, via job na partição gpu):
srun -p gpu --gres=gpu:1 nvidia-smi
```

Interpretação rápida do `sinfo -N -l`: a coluna `S:C:T` é *Sockets : Cores por socket : Threads por core* (ex.: `2:6:2` = 24 CPUs lógicas), e `MEMORY` é a RAM disponível para jobs, em MB.

### 4.1 Lendo os estados dos nós

Exemplo de saída do `sinfo`:

```
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
normal*      up 3-00:00:00      2  down* c[1,5]
normal*      up 3-00:00:00      7   idle c[2-4,6-7,9-10]
gpu          up 3-00:00:00      1   unk* c11
```

| Elemento | Significado |
|----------|-------------|
| `normal*` | O asterisco no nome marca a **partição padrão** (usada quando você não informa `--partition`). |
| `3-00:00:00` | Tempo máximo por job: 3 dias = 72 h. |
| `idle` | Nó livre, pronto para receber jobs. |
| `alloc` / `mix` | Nó ocupado (totalmente / parcialmente). |
| `down` | Nó fora de operação (falha ou marcado pelo administrador). |
| `drain` | Nó não aceita novos jobs (manutenção programada); os jobs em andamento terminam. |
| `unk` (unknown) | O controlador **nunca recebeu resposta** do `slurmd` desse nó — típico de nó ainda não integrado. |
| Sufixo `*` no estado | O nó não está respondendo ao controlador no momento. |

No exemplo acima: 7 nós disponíveis na `normal`, c1 e c5 fora do ar, e o nó de GPU (c11) ainda não integrado. Submeter para uma partição sem nós operacionais deixa o job em `PD` (pendente) por tempo indeterminado — verifique o `sinfo` antes.

---

## 5. Instalando bibliotecas (sem sudo)

Usuários **não têm sudo**, e não precisam. Bibliotecas Python (e similares) são instaladas no **seu diretório home**, que é compartilhado com todos os nós.

**Importante: instale sempre a partir do nó de acesso.** Os nós de computação não têm acesso à internet; `pip install` executado dentro de um job falhará ao baixar pacotes.

### Opção A — ambiente virtual Python (recomendado)

```bash
# No nó de acesso, uma única vez:
python3 -m venv ~/venvs/meuprojeto
source ~/venvs/meuprojeto/bin/activate
pip install --upgrade pip
pip install numpy scipy matplotlib
```

Nos seus scripts de job, ative o ambiente antes de executar (veja o exemplo de `sbatch` na seção 6).

### Opção B — pip --user

```bash
pip3 install --user nome-do-pacote
```

Instala em `~/.local/`. Funciona, mas o venv (Opção A) isola melhor os projetos.

### Opção C — Miniconda (para quem prefere conda)

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh -b -p ~/miniconda3
~/miniconda3/bin/conda init bash
# saia e entre novamente, depois:
conda create -n meuprojeto python=3.11
conda activate meuprojeto
```

### Caso especial: PyTorch (torch)

Duas observações importantes:

1. **As CPUs da partição `normal` são de geração antiga (sem AVX2).** As versões recentes do PyTorch exigem AVX2 e falham nesses nós com *"Illegal instruction"*. **Use o PyTorch na partição `gpu`** (nó c11, CPU moderna + RTX 3060).
2. Para usar a GPU, instale a variante com CUDA:

```bash
source ~/venvs/meuprojeto/bin/activate
pip install torch --index-url https://download.pytorch.org/whl/cu121
```

E rode na partição `gpu`:

```bash
srun -p gpu --gres=gpu:1 python -c "import torch; print(torch.cuda.is_available())"
```

Se imprimir `True`, o PyTorch está enxergando a GPU.

> Enquanto o nó c11 estiver em integração (ver seção 1), esse teste não retorna: o job fica pendente na fila. Uma alternativa provisória para código que roda em CPU é instalar a variante CPU-only de uma versão anterior do PyTorch, compatível com máquinas sem AVX2 — consulte o suporte.

---

## 6. Executando jobs

### Job interativo (para testar)

```bash
# Um shell em um nó de computação:
srun --pty bash

# Um comando rápido em um nó:
srun -N1 hostname

# Shell no nó com GPU:
srun -p gpu --gres=gpu:1 --pty bash
```

### Job em lote (sbatch) — o modo normal de trabalhar

Crie um arquivo, por exemplo `meujob.sh`:

```bash
#!/bin/bash
#SBATCH --job-name=simulacao
#SBATCH --partition=normal
#SBATCH --nodes=1
#SBATCH --time=24:00:00
#SBATCH --output=%x-%j.out

source ~/venvs/meuprojeto/bin/activate
python simulacao.py
```

Submeta e acompanhe:

```bash
sbatch meujob.sh      # submete (retorna o número do job)
squeue                # fila: seu job aparece como R (rodando) ou PD (aguardando)
scancel NUMERO        # cancela um job seu
```

A saída do job fica no arquivo `simulacao-NUMERO.out`.

### Job com GPU

*(Válido quando o nó c11 estiver operacional — ver seção 1.)*

```bash
#!/bin/bash
#SBATCH --job-name=treino
#SBATCH --partition=gpu
#SBATCH --gres=gpu:1
#SBATCH --time=48:00:00
#SBATCH --output=%x-%j.out

source ~/venvs/meuprojeto/bin/activate
python treino.py
```

### Limites e comportamento

- **Tempo máximo por job: 72 h** (`--time` não pode exceder isso). Para trabalhos mais longos, use *checkpointing* e divida em etapas.
- Na partição `normal`, cada job ocupa **um nó inteiro**; aproveite as 24 CPUs (paralelize).
- **SSH direto aos nós de computação é bloqueado**, exceto se você tiver um job ativo naquele nó. Use `srun --pty bash` para sessões interativas.

---

## 7. Armazenamento

| Local | O que é | Características |
|-------|---------|-----------------|
| `/home/seu.usuario` | Seu diretório pessoal | Compartilhado entre todos os nós (NFS). Onde ficam códigos, ambientes e resultados. |
| `/scratch` | Disco local de cada nó | Rápido, ~925 GB, **local a cada nó** (o conteúdo não é visível de outros nós). Para arquivos temporários de jobs. A variável `TMPDIR` já aponta para ele automaticamente. |

Boas práticas:

- `/home` é um recurso **compartilhado**; evite acumular centenas de GB e limpe resultados antigos.
- `/scratch` **não tem backup** e pode ser limpo em manutenções; não deixe nada importante lá.
- Faça backup dos seus resultados importantes para fora do cluster (o cluster não substitui backup pessoal).

---

## 8. Boas práticas e etiqueta

1. **Nada de processamento pesado no nó de acesso**: ele é de todos. Compilou? Testou 30 segundos? Ok. Rodar uma simulação de horas nele? Não, use `sbatch`.
2. **Dimensione o `--time` com honestidade**: jobs com estimativa realista escalonam melhor para todo mundo.
3. **Cancele jobs que não precisa mais** (`scancel`); isso libera os nós para os colegas.
4. **A partição `gpu` é para trabalhos que usam GPU**; jobs somente-CPU devem ir para a `normal`.
5. Problemas, dúvidas ou pedidos de instalação de software de sistema: contate o suporte.

---

## 9. Referência rápida

```bash
ssh lftc                          # conectar
scp -r projeto/ lftc:~/           # enviar arquivos
sinfo -N -l                       # ver nós e specs
squeue                            # ver fila
sbatch meujob.sh                  # submeter job
scancel 123                       # cancelar job 123
srun --pty bash                   # shell interativo em um nó
srun -p gpu --gres=gpu:1 --pty bash   # shell no nó GPU
```

Acesso gráfico por SFTP (arrastar e soltar): `sftp://150.161.8.51/home/seu.usuario/` — ver seção 3.1.

---

*Documento mantido pelo suporte do cluster (NLINK). Sugestões de melhoria são bem-vindas.*
