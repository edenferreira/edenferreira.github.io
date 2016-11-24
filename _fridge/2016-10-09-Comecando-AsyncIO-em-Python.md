---
layout: post
title: Começando AsyncIO em Python - Aventuras com Python Async
author: Éden Thiago Ferreira
---

Até hoje eu só tinha feita async usando javascript/node, e lá é "de graça". Você tem que pensar de forma assíncrona, e se preocupar em como organizar o código de forma a não se perder, como combinar eventos acontecendo em momentos arbitrários, porém a base da assíncronicidade já está feita para você.

Por exemplo, para ler uma arquivo de forma assíncrona em node:
{% highlight javascript %}
const fs = require('fs');

fs.readFile('algum_arquivo.bla', (err, res) => {
    doSomething(res);
});
{% endhighlight %}

Em imaginei que fosse uma api própria do OS (Como se fala, exportada?), mas quando fui repetir isso em python, tentando não usar bibliotecas de terceiros, descobri que para fazer isso é necessário que a leitura seja feita em uma thread se eu quiser que a leitura não bloqueie a thread principal.

O node esconde isso, eu uso a um bom tempo que nunca pensei que fosse uma thread separada (menos curiosidade do que deveria? Provavelmente). Um dos meus zen do python preferios é *Implícito é melhor que explícito*, então pra fazer a leitura de arquivo sem bloquear _EU_ tive que usar thread pela primeira vez em python. (Quem sabe o que algum framework que usei já fez. Falta de curiosidade²).

Todos os exemplos estarão em python 3.5. Aqui eu usei uma async.io e ThreadPoolExecutor para usar o readline do arquivo de forma assíncrona. Também adicionei funções no meio para mostrar que a execução é realmente assíncrona:
{% highlight python %}
import asyncio
from concurrent.futures import ThreadPoolExecutor

executor = ThreadPoolExecutor()
loop = asyncio.get_event_loop()

async def read(fileName):
    lines = []
    with open(fileName) as f:
        while True:
            await asyncio.sleep(1)
            line = await loop.run_in_executor(
                executor, f.readline)
            if line:
                print('line', line)
                lines.append(line)
            else:
                return lines

#function just to mess things up
async def just_to_wait(id):
    wait_in_seconds = 10 - id
    await asyncio.sleep(wait_in_seconds)
    print('wait:', id, 'seconds:', wait_in_seconds)

loop = asyncio.get_event_loop()

result = loop.run_until_complete(asyncio.gather(
    read('file'),
    just_to_wait(1),
    just_to_wait(2),
    just_to_wait(3),
    just_to_wait(4),
    just_to_wait(5),
    just_to_wait(6),
    just_to_wait(7),
))
print(result[0]) #just the result of the read
loop.close()
{% endhighlight %}

Felizmente o retorno desse script foi o esperado:
{% highlight bash %}
python3 experiment.py 
line linha 1

line linha 2 

wait: 7 seconds: 3
line linha 3

wait: 6 seconds: 4
line linha 4

wait: 5 seconds: 5
line linha 5

wait: 4 seconds: 6
line linha 6

wait: 3 seconds: 7
line linha 7

wait: 2 seconds: 8
line linha 8

wait: 1 seconds: 9
line linha 9
['linha 1\n', 'linha 2 \n', 'linha 3\n', 'linha 4\n', 'linha 5\n', 'linha 6\n', 'linha 7\n', 'linha 8\n', 'linha 9']
{% endhighlight %}

Note o '\n' depois e cada linha. Esse não era um arquivo muito criativo. Mas você pode notar que a leitura das linhas foi individual e não bloqueou o a execução da thread principal.

Minha próxima aventura será usar o que aprendi com esse código e criar um iterable assíncrono das linhas dos arquivos.