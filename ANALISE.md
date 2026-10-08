# Observação

Nos prints, eu formatei alguns para melhor visualização do conteúdo. Inclusive já deixando o prefixo >>> LOOP, >>> ACTION e >>> THOUGH para exemplificar minha análise.

# Análise

### 1. Loop:

Deixei no trace.log uma das execuções em que o agente conseguiu resolver a tarefa em um loop de iteração, mas houve outras que eu precisei re-enviar o prompt incial para uma nova iteração loop. Normalmente a saída desses laços se dava quando ele não conseguia identificar a chamada da tool corretamente.

```
>>> ACTION: Iniciando o loop do agente de codificação.
>>> LOOP:  0
(...)
>>> ACTION: voltando ao loop principal para nova entrada do usuário
>>> LOOP:  1
```

### 2. Contexto:

O resultado da tool é adicionado na variável conversation. No início de tudo, também, foi adicionado a chamada incial do sistema passando o que o agente representa e as tools disponíveis, depois o prompt do usuário e ao longo da cadeia de pensamento as chamadas e resultados das tools invocadas.

```
>>> CONTEXTO:   [
    {
        "role": "system",
        "content": "\nYou are a coding assistant whose goal it is to help us solve coding tasks.\nYou can perform actions by emitting a single command line in exactly this format, and nothing else on that line:\n\ntool: NAME({\"arg\": \"value\"})\n\nDo not use JSON function-calling, a <tool_call> tag, or any other structured tool-call format your training may default to.\nThe ONLY format the system running you understands is the plain text line above.\n\nAvailable commands:\n\nTOOL\n===\n    Name: read_file\n    Description: \n    Gets the full content of a file provided by the user.\n    :param filename: The name of the file to read.\n    :return: The full content of the file.\n    \n    Signature: (filename: str) -> Dict[str, Any]\n    \n===============\nTOOL\n===\n    Name: list_files\n    Description: \n    Lists the files in a directory provided by the user.\n    :param path: The path to a directory to list files from.\n    :return: A list of files in the directory.\n    \n    Signature: (path: str) -> Dict[str, Any]\n    \n===============\nTOOL\n===\n    Name: edit_file\n    Description: \n    Replaces first occurrence of old_str with new_str in file. If old_str is empty,\n    create/overwrite file with new_str.\n    :param path: The path to the file to edit.\n    :param old_str: The string to replace.\n    :param new_str: The string to replace with.\n    :return: A dictionary with the path to the file and the action taken.\n    \n    Signature: (path: str, old_str: str, new_str: str) -> Dict[str, Any]\n    \n===============\n\n\nExample of a correct response when you want to read a file named 'notes.txt':\ntool: read_file({\"filename\": \"notes.txt\"})\n\nUse compact single-line JSON with double quotes. After receiving a tool_result(...) message, continue the task using the same format when another action is needed.\nIf no action is needed, respond in plain prose.\n"
    },
    {
        "role": "user",
        "content": "encontre e conserte o bug baseado no teste que está falhando em test_inventory.py"
    },
    {
        "role": "user",
        "content": "tool_result({\"path\": \"/home/anarita/ufcg/devia/atividade-anatomia-agentes\", \"files\": [{\"filename\": \"README.md\", \"type\": \"file\"}, {\"filename\": \".git\", \"type\": \"dir\"}, {\"filename\": \"requirements.txt\", \"type\": \"file\"}, {\"filename\": \"agent.py\", \"type\": \"file\"}, {\"filename\": \"inventory.py\", \"type\": \"file\"}, {\"filename\": \"trace.log\", \"type\": \"file\"}, {\"filename\": \".gitignore\", \"type\": \"file\"}, {\"filename\": \"test_inventory.py\", \"type\": \"file\"}, {\"filename\": \".env\", \"type\": \"file\"}]})"
    },
    {
        "role": "user",
        "content": "tool_result({\"file_path\": \"/home/anarita/ufcg/devia/atividade-anatomia-agentes/test_inventory.py\", \"content\": \"from inventory import apply_discount\\n\\n\\ndef test_apply_discount():\\n    assert apply_discount(200, 10) == 180\\n\"})"
    },
    {
        "role": "user",
        "content": "tool_result({\"file_path\": \"/home/anarita/ufcg/devia/atividade-anatomia-agentes/inventory.py\", \"content\": \"def apply_discount(price, percent):\\n    \\\"\\\"\\\"Aplica um desconto percentual ao pre\\u00e7o.\\\"\\\"\\\"\\n    return price - percent\\n\"})"
    },
    {
        "role": "user",
        "content": "tool_result({\"path\": \"/home/anarita/ufcg/devia/atividade-anatomia-agentes/inventory.py\", \"action\": \"edited\"})"
    }
]
```

### 3. Tools/ACI:

Uma das chamadas das tools disponíveis foi a de edição do arquivo, para corrigir o problema.

A chamada feita por mim usuário:

```
You::encontre e conserte o bug baseado no teste que está falhando em test_inventory.py
```

Como foi parseada para o contexto:

```
{
  "role": "user",
  "content": "encontre e conserte o bug baseado no teste que está falhando em test_inventory.py"
}

```

Chamada de tool pelo agente:

```
>>> ACTION: invocando a tool  read_file {'filename': 'test_inventory.py'}
```

Ambas são estruturadas em um Json chave valor, uma linguagem mais estruturada pro agente entender e a chamada das tools segue a estrutura nome_da_tool {nome_arg: valor_arg}.

### 4.Thought:

Nessa execução que fiz, o agente raciocinou que o problema inicial teve o bug corrigido e que está passando nos testes, então ele retorna pro usuário que o bug foi corrigido e logo depois verifica que não há mais ferramentas pra serem chamadas e volta para a iteração do loop principal.

````
>>> THOUGHT: Bug corrigido.

**Problema:** Em `inventory.py`, a função `apply_discount` subtraía o valor percentual diretamente do preço em vez de calcular o desconto percentual. Com `apply_discount(200, 10)`, o resultado era `200 - 10 = 190`, mas o teste espera `180`.

**Correção:**

```python
def apply_discount(price, percent):
    """Aplica um desconto percentual ao preço."""
    return price - (price * percent / 100)
```

Agora `apply_discount(200, 10)` retorna `200 - (200 * 10 / 100) = 200 - 20 = 180`, passando no teste.
Assistant:: Bug corrigido.

**Problema:** Em `inventory.py`, a função `apply_discount` subtraía o valor percentual diretamente do preço em vez de calcular o desconto percentual. Com `apply_discount(200, 10)`, o resultado era `200 - 10 = 190`, mas o teste espera `180`.

**Correção:**

```python
def apply_discount(price, percent):
    """Aplica um desconto percentual ao preço."""
    return price - (price * percent / 100)
```

Agora `apply_discount(200, 10)` retorna `200 - (200 * 10 / 100) = 200 - 20 = 180`, passando no teste.

>>> THOUGHT: não há mais ferramentas a invocar
>>> ACTION: voltando ao loop principal para nova entrada do usuário
````

### 5. Guardrail

Sim, tirando as paradas por não ter a chamada de tool com o prefixo esperado `tool:`, ele parou achando que resolveu. Em uma das execuções que eu fiz isso aconteceu, ele entendia que havia resolvido o problema, mas quando eu lançava o prompt inicial novamente, ele notou que ainda havia persistência do bug, aplicou correção e rodou novamente o teste para validar.

### Falhas de parsing

Não ocorreu nesse trace.log final que eu deixei, pois foi uma execução que considerei positiva e preferi registrar ela, mas em outras ele deixou passar chamadas de tools que não estavam no formato esperado. Nesses casos, ele retornou ao loop principal esperando uma nova entrada por mim.
