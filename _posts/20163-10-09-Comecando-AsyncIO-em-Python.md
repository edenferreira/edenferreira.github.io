---
layout: post
title: Começando AsyncIO em Python - Aventuras com Python Async
author: Éden Thiago Ferreira
---

Até hoje eu só tinha feita async usando javascript/node, e lá é "de graça". Você tem que pensar de forma assíncrona, e se preocupar em como organizar o código de forma a não se perder, como combinar eventos acontecendo em momentos arbitrários, porém a base da assíncronicidade já está feita para você.

Por exemplo, para ler uma arquivo de forma assíncrona em node:
    const fs = require('fs');
    
    fs.readFile('algum_arquivo.bla', (err, res) => {
        doSomething(res);
    });

Em imaginei que fosse uma api que o próprio OS (exportava?), mas quando fui repetir isso em python, tentando não usar bibliotecas de terceiros, descobri que para fazer isso é necessário que a leitura seja feita em uma thread se eu quiser que a leitura não bloqueie a thread principal.

O node esconde isso, eu uso a um bom tempo que nunca pensei que fosse uma thread separada (menos curiosidade do que deveria? Provavelmente). Um dos meus zen do python preferios é *Implícito é melhor que explícito*, então pra fazer a leitura de arquivo sem bloquear _EU_ tive que usar thread pela primeira vez em python. (Quem sabe o que algum framework que usei já fez. Falta de curiosidade²).

Todos os exemplos estarão em python 3.5. Aqui eu usei uma async.io e ThreadPoolExecutor para usar o readline do arquivo de forma assíncrona. Também adicionei funções no meio para mostrar que a execução é realmente assíncrona:
    import asyncio
    from concurrent.futures import ThreadPoolExecutor
    
    executor = ThreadPoolExecutor()
    loop = asyncio.get_event_loop()
    
    async def read(fileName):
        lines = []
        with open(fileName) as f:
            while True:
                await asyncio.sleep(1)
                line = await loop.run_in_executor(executor, f.readline)
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

eu até hoje só fiz async de verdade em node, e lá tudo é async, então tive q pensar em como trabalhar com async mas não como certas coisas funcionam por trás.
python é mais explícito, faz parte do zen, algo q eu gosto, e assim descrobri q ler e escrever um arquivo async não é "de graça" e precisa de uma thread.
sem problemas, a primeira parte e meia das minhas aventuras no async python vai ser fazer um objeto q le um arquivo de forma assíncrona usando um executor q usarei na continuação.

Depois de experimentar algumas coisas e ler algumas docuIentações, SO e exemplos cheguei a isso:

criei uma função q fica dormindo para ver a leitura realmente assíncrona, o resultado da execução disso é:

Agora q entendi melhor como funciona isso, vamos fazer um objeto para ler o arquivo de forma assíncrona.
Começando com um iterable simples, já recebendo o arquivo, além do loop e o executor, como boa inversão de dependência dita.
Mas isso fica meio chato né, então continuando no async criei um context manager assíncrono para cuidar de abrir e fechar o arquivo de maneira correta, e apenas retorna o iterable para fazermos tudo q temos q fazer, um exemplo de uso no final.
"Próximo teste unitário?"