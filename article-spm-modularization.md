Modularização no iOS: Estruturando Projetos com Swift Package Manager

Ajudando em reutilização, escalabilidade e gerenciamento de dependências, é comum que projetos mobile sejam modularizados.
Pense em um app financeiro, como exemplo esse abaixo [achado numa plataforma chamada dribbble](https://dribbble.com/following)

![Screenshot de um projeto de aplicativo financeiro](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hlboc96zuxxh31j1llmw.png)

Pensando em times trabalhando em diferentes partes do app, modularizar ajudaria cada equipe a cuidar de um módulo específico. Aqui poderíamos ter um módulo para "transfer", "invest" e "budget", e eles seriam instanciados em um projeto principal. Também pensando em reutilização de código, é comum que haja um módulo para a parte de design system e componentes que serão utilizados. Ajudam na testabilidade, padronização do projeto e também evitará a repetição de código, já que cada módulo do projeto poderá ter o módulo de componentes.

![Diagrama módulos](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wl0uigv6v86piubz0r8c.png)

Uma possível estrutura:

- BankApp seria nosso projeto principal, adicionaremos dependências nele.
- Invest, Transfer e Budget serão dependências de BankApp.
- Components será uma dependência de Invest, Transfer e Budget.
- Firebase* e Alamofire* também serão dependências de BankApp.

*Como são libs mais conhecidas, quis exemplificar em que "nível" ela pode estar quando for usada no app principal.

E também não é uma regra, já que você pode criar módulos que tenham o Alamofire, por exemplo, e, nesse caso, o app principal não precisaria conhecê-lo.

O [Cocoapods](https://github.com/CocoaPods/CocoaPods) e o [Carthage](https://github.com/Carthage/Carthage) são outras formas de trabalhar com gerenciamento de dependencias e distribuição de bibliotecas em um projeto iOS, mas pra esse projeto, usaremos [o Swift Package Manager](https://www.swift.org/documentation/package-manager/)

Uma curiosidade é que ele foi anunciado pela primeira vez em 2015, é open source e está disponível no [repositório do Swift no GitHub](https://github.com/swiftlang/swift-package-manager). Trabalhar com ele é bom por essa experiência mais nativa e integrada ao xcode. 

Criando um projeto

Abra seu xcode

File > New > Project

![Opções de novo projeto no Xcode](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1qkkmz1vuxb9zwgll3tt.png)

![Configurando um novo projeto no Xcode](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ippx5jyouivndhtn4ymy.png)

![Screenshot criando projeto com nome finance-app](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1pw458kiq8do5r9ropr5.png)

Repita esse processo, agora criando um projeto para o que será nosso módulo, o nome do projeto pode ser Components

Quando criado, acessei a pasta onde estava meu projeto 

```swift
cd Components
```

**Primeiros comandos para instanciar o spm no seu diretório**

Pra inicializar um Package.swift e /Sources no diretório atual.

```swift
swift package init --type executable
```

Pra buscar e resolver todas as dependências especificadas no arquivo Package.swift do projeto.


```swift
swift package resolve
```

Pra compilar o código-fonte do pacote e todas as suas dependências. 

```swift
swift build
```

**Entendendo o package.swift**

O nome do pacote, que deve ser único dentro do contexto de onde ele será usado.

```swift
import PackageDescription

let package = Package(
    name: "components-views",
```

Especifica as plataformas e versões mínimas suportadas pelo pacote.

```swift
platforms: [
        .macOS(.v10_15),
        .iOS(.v13),
        .tvOS(.v13),
        .watchOS(.v6)
    ],
```

Produtos são as bibliotecas ou executáveis que o pacote produz e pode compartilhar com outros pacotes. Nesse caso, quando nosso projeto utilizar esse pacote, poderá importar Components pra acessar o que há no target Components, isso é útil porque podemos separar os targets que serão exibidos, e o projeto que utilizar esse módulo, poderá escolher qual library ele irá importar. 
   
```swift
products: [
        .library(
            name: "Components",
            targets: ["Components"]),
    ],
```
Permite especificar dependências externas, como outros pacotes Swift disponíveis em repositórios Git.

```swift
dependencies: [
        .package(url: "https://github.com/pointfreeco/swift-snapshot-testing", from: "1.10.0")
    ],
```

Targets são as unidades de construção do pacote. Podem ser módulos, bibliotecas ou executáveis.
    
```swift    
    targets: [
        .target(
            name: "Components",
            dependencies: []),
        .testTarget(
            name: "ComponentsTests",
            dependencies: ["Components"]),
        .testTarget(
            name: "ComponentsUITests",
            dependencies: ["Components"]),
    ]
)

```

Para conseguirmos visualizar pelo menos um ínicio de tela, crie em Components uma view chamada TransactView

```swift
import SwiftUI

public struct TransactView: View {
    
    public init() { }
    
    public var body: some View {
        VStack {
            Text("Transact")
                .font(.headline)
            
            HStack(spacing: 16) {
                createImageView(with: "Buy", imageName: "envelope")
                createImageView(with: "Sell", imageName: "scissors")
                createImageView(with: "Deposit", imageName: "arrow.down")
                createImageView(with: "Withdraw", imageName: "arrow.up")
                createImageView(with: "More", imageName: "plus")
            }
            .padding()
        }
        .padding()
    }
    
    func createImageView(with text: String, imageName: String) -> some View {
        VStack {
            Image(systemName: imageName)
                .padding()
                .background(Circle().fill(Color.gray.opacity(0.2)))
            Text(text)
                .font(.caption)
                .multilineTextAlignment(.center)
        }
        .frame(width: 60)
    }
}

```

Agora, crie um repositório público [no github](https://github.com/) e publique seu diretório criado, vamos deixar ele visível para adicionarmos no app principal.

Depois de criado, abra o projeto do app principal e vá até a raiz do projeto.

![Menu com as pastas que o projeto finance-app tem](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xioazm0moomtzo4btsrm.png)

Clique em package dependencies, e depois, no botão + mais abaixo.

![Screenshot da tela quando clicado em package dependencies](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/cerxk74f8kfemnprxv4f.png)

Coloque o link do seu repositório e veja que você não precisa utilizar ele só da main, há outras opções também.

![Adicionando dependência por branch no Xcode](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9dy1k77w50c25c3lbqey.png)

Clique em Add Package, e quando for adicionado com sucesso, a estrutura do Components deve se parecer com essa imagem.

![Estrutura do package dentro do finance-app](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dfs448crdzk5756kb5el.png)

Agora, ja conseguimos importar Components

![Screenshot da tela no xcode com o arquivo FinanceApp aberto](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tc0cwnq2868v8f3ah11b.png)

Deixaremos nossa ContentView assim

```swift
import SwiftUI
import Components

struct ContentView: View {
    var body: some View {
        TransactView()
    }
}

#Preview {
    ContentView()
}
```
E ao clicar em run, quando o simulador for aberto, a tela deve se parecer com esse componente

![Simulação de Iphone 12 mostrando o componente de transações, uma lista de imagens com os nomes "buy", "sell", "deposit", "withdraw" e "more"](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/q61zbiy75ip15zpk6e8o.png)

***Adicionando módulos localmente***

Sem a necessidade de criar repositórios, no xcode há a possibilidade de adicionar um package localmente.

No projeto, vá até

File > Add Package Dependencies

Clique em Add Local e depois de escolher o diretório do seu módulo, a sua tela deve parecer isso.

![Screenshot da tela de selecionar target que usará o módulo adicionado](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8cgzow60t6w8al7ykqj2.png)

Certifique-se de não estar com o projeto do seu módulo aberto numa outra aba do xcode, isso pode interferir no build e seu módulo não será carregado. 
