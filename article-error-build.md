Resolvendo erro de build no projeto de arquitetura Intel para arquitetura Apple Silicon (M1)

Resolvendo "error: module was created for incompatible target arm64-apple-ios9.0"

Meu primeiro dia de trabalho foi bem interessante, ainda não tínhamos testado o projeto em um computador Mac com Apple Silicon (M1), com isso, tivemos um erro subindo o projeto. 

Uma informação que nos ajudará a entender o problema é que, com essa nova plataforma teremos o macOS baseado em arm64, seus simuladores também serão executados em arm64, ao contrário da atual arquitetura x86/64 baseada na Intel.

O projeto do trabalho, que havia sido criado com arquitetura x64, tem algumas dependências que não suportam a nova arquitetura dos MacBooks, então foi preciso mexer em uma configuração.

Como podemos ver abaixo, foi assim que ficou o **Build Settings** após fazer as alterações, fomos em **Excluded Architectures**, depois em **Debug** e **Release** adicionamos a arquitetura que queremos excluir quando subirmos o projeto no simulador **(Any iOS Simulator)**.

![Um código excluindo a arquitetura arm64 quando rodar o projeto no simulador](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/s9pjss7qo0hod54qf8jf.png)

Nesse vídeo eu mostro como configurei, porém em outro projeto que estava mexendo. 

https://www.youtube.com/watch?v=l3Cmcyqbvxg

Lembrando que eu não rodo o projeto em um celular, e sim no simulador do Xcode mesmo.

Fazendo isso, pode acontecer com que o build fique mais lento, porque ele praticamente faz a transcrição do código com a arquitetura x64 (Intel) para a arm64 (Apple Silicon), mas isso resolveu o problema, já que iremos evitar as dependências que não suportam essa nova arquitetura de nos impedir de subir o projeto. 

O M1 consegue executar códigos feitos na arquitetura Intel com o Rosetta 2. Um link interessante sobre isso [aqui](https://eclecticlight.co/2021/01/22/running-intel-code-on-your-m1-mac-rosetta-2-and-oah/) 

E um agradecimento final ao meu líder que conseguiu me explicar isso pacientemente e me ajudar a solucionar o problema. Obrigada, Felipe!
