
I have been working with web apps for most of my career, a few exceptions are under my belt, but mostly web applications. The web is a world by itself and one can not hope to know everything about the topic. There are layers and layers of abstractions to make a single page work, some are straighforward, some are hands down complex, my experience is on scaling medium size web applications with Python and Javascript. No Facebook-like applications to scale, no need for global orchestration, just managable apps of growing start-ups.

Until I discovered a new type of web applications: Decentralized Apps (Dapps).

Dapps are a growing movement of applications built on top of the Ethereum blockchain with the goal of making them:
- Decentralized
- Accessible by anyone
- No owned by corporations (or even anyone)
- Free from censorship
- Private
- With no downtime

DApps are pure frontend applications in the sense that you don't hold a server with the global state of your application, all connections are made directly to the blockchain to fetch information and initiate transactions. The Ethereum blockchain is used for data storage and smart contracts as the app logic. The consequence of this arrengement is that once dapps are deployed to the Ethereum network, they can not be change, assuring that the only thing that controls the application is the logic written into the contract and not an individual or a company. A smart contract is a program that runs on the Ethereum blockchain, imagine classes that represent your domain models and methods that perform actions on these models. User accounts can interact with a smart contract by submitting transactions that execute functions of the smart contract, besides contracts can define rules and automatically enforce them via the code.

In this post I intent to discuss some of the high-level differences of this type of application, a later post will address the differnt technical concepts and how everything fits together.

Blockchain as a utility



A few weeks of work, showed me how interest and vast the web is.

Differently from regular apps where we use service providers for deployment (such as Heroku), these type of applications are usually hosted on decentralized storage such as IPFS (details on this later). About the backend, DApps use smart contracts as their backend.



All these information refers to a change on the way that the web works, Dapps propose paradigm shift on the web, from web2 to web3. Web2 refers to the version of the internet most of us know nowadays. An internet dominated by companies that provide services in exchange for your personal data. Web3, in the context of Ethereum, refers to decentralized apps that run on the blockchain. These are apps that allow anyone to participate without monetizing their personal data.


IPFS A peer-to-peer hypermedia protocol
