# iPag PHP client SDK
> A ferramenta certa para uma rápida e segura integração com o iPag e a sua aplicação PHP

> SDK Status

[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/jhernandes/ipag-sdk-php/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/jhernandes/ipag-sdk-php/?branch=master)
[![Maintainability](https://api.codeclimate.com/v1/badges/a2ca7f797ac6f1084129/maintainability)](https://codeclimate.com/github/jhernandes/ipag-sdk-php/maintainability)
[![StyleCI](https://styleci.io/repos/115621915/shield?branch=master)](https://styleci.io/repos/115621915)
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/1c91d5c5f9a7465bae8ad4c3ee33f232)](https://www.codacy.com/app/jhernandes/ipag-sdk-php?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=jhernandes/ipag-sdk-php&amp;utm_campaign=Badge_Grade)
[![Codacy Badge](https://api.codacy.com/project/badge/Coverage/1c91d5c5f9a7465bae8ad4c3ee33f232)](https://www.codacy.com/app/jhernandes/ipag-sdk-php?utm_source=github.com&utm_medium=referral&utm_content=jhernandes/ipag-sdk-php&utm_campaign=Badge_Coverage)
[![Build Status](https://scrutinizer-ci.com/g/jhernandes/ipag-sdk-php/badges/build.png?b=master)](https://scrutinizer-ci.com/g/jhernandes/ipag-sdk-php/build-status/master)
---

**Índice**

- [Dependências](#dependências)
- [Instalação](#instalação)
- [Autenticação](#autenticação)
    - [Por Basic Auth](#por-basic-auth)
- [Cliente](#cliente)
    - [Dados do Cliente](#dados-do-cliente)
- [Cartão de Crédito/Débito](#cartão-de-crédito/débito)
    - [Dados do Cartão de Crédito/Débito](#dados-do-cartão-de-crédito/débito)
- [Carrinho](#carrinho)
    - [Adicionando Produtos](#adicionando-produtos)
- [Transação](#transação)
    - [Com Cartão de Crédito](#transação-com-cartão-de-crédito)
    - [Com Token de Cartão de Crédito](#transação-com-token-de-cartão-de-crédito)
    - [Com Boleto](#transacao-com-boleto)
    - [Consulta](#consulta)
    - [Captura](#captura)
    - [Captura Parcial](#captura-parcial)
    - [Cancelamento](#cancelamento)
    - [Cancelamento Parcial](#cancelamento-parcial)
- [Assinatura](#assinatura)
    - [Criando uma Assinatura](#criando-uma-assinatura)
- [Exemplo de Transação Completa](#exemplo-de-transação-completa)
- [Exemplo de Transação com Regra de Split](#exemplo-de-transacao-com-regra-de-split)
- [Exemplo de Página de Callback](#exemplo-de-página-de-callback)
- [Resposta](#responta)
- [Testes](#testes)
- [Licença](#licença)
- [Documentação](#documentação)
- [Dúvidas & Sugestões](#duvidas-&-sugestões)

## Dependências

**require**
 - [PHP >= 5.6]

**require-dev**
 - [phpunit/phpunit]
 - [codacy/coverage]

## Instalação

Execute em seu shell:

    composer require jhernandes/ipag-sdk-php

## Autenticação
### Por Basic Auth

```php
require 'vendor/autoload.php';

use Ipag\Ipag;
use Ipag\Classes\Authentication;
use Ipag\Classes\Endpoint;

$ipag = new Ipag(new Authentication('my_id_ipag', 'my_ipag_key'), Endpoint::SANDBOX);
```
## Cliente
### Dados do Cliente

```php
$customer = $ipag->customer()
    ->setName('Fulano da Silva')
    ->setTaxpayerId('799.993.388-01')
    ->setPhone('11', '98888-3333')
    ->setEmail('fulanodasilva@gmail.com')
    ->setBirthdate('1989-03-28')
    ->setAddress($ipag->address()
        ->setStreet('Rua Júlio Gonzalez')
        ->setNumber('1000')
        ->setNeighborhood('Barra Funda')
        ->setCity('São Paulo')
        ->setState('SP')
        ->setZipCode('01156-060')
);
```

## Cartão de Crédito/Débito
### Dados do Cartão de Crédito/Débito

```php
$creditCard = $ipag->creditCard()
    ->setNumber('4066553613548107')
    ->setHolder('FULANO')
    ->setExpiryMonth('10')
    ->setExpiryYear('2025')
    ->setCvc('123')
    ->setSave(true); //True para gerar o token do cartão (one-click-buy)
```

## Carrinho
### Adicionando Produtos

```php
// ...

$cart = $ipag->cart(
    // Nome do Produto, Valor Unitário, Quantidade, SKU (Código do Produto)
    ['Produto 1', 5.00, 1, 'ABDC1'],
    ['Produto 2', 3.50, 2, 'ABDC2'],
    ['Produto 3', 5.50, 1, 'ABDC3'],
    ['Produto 4', 8.50, 5, 'ABDC4']
);

// ...
```
## Transação (Pagamento)
### Transação com Cartão de Crédito
```php
$transaction = $ipag->transaction();

$transaction->getOrder()
    ->setOrderId($orderId)
    ->setCallbackUrl('https://minha_loja.com.br/ipag/callback')
    ->setAmount(10.00)
    ->setInstallments(1)
    ->setPayment($ipag->payment()
        ->setMethod(Method::VISA)
        ->setCreditCard($creditCard)
    )->setCustomer($customer)
);

$response = $transaction->execute();
```

### Transação com Token de Cartão de Crédito
```php
$transaction = $ipag->transaction();

$transaction->getOrder()
    ->setOrderId($orderId)
    ->setCallbackUrl('https://minha_loja.com.br/ipag/callback')
    ->setAmount(10.00)
    ->setInstallments(1)
    ->setPayment($ipag->payment()
        ->setMethod(Method::VISA)
        ->setCreditCard($ipag->creditCard()
            ->setToken('ABDC-ABCD-ABCD-ABDC')
        )
    )->setCustomer($customer)
);

$response = $transaction->execute();
```

### Transação com Boleto
```php
$transaction = $ipag->transaction();

$transaction->getOrder()
    ->setOrderId($orderId)
    ->setCallbackUrl('https://minha_loja.com.br/ipag/callback')
    ->setAmount(10.00)
    ->setInstallments(1)
    ->setExpiry('10/10/2017')
    ->setPayment($ipag->payment()
        ->setMethod(Method::BANKSLIP_ZOOP)
    )->setCustomer($customer)
);

$response = $transaction->execute();
```

### Consulta

```php
$response = $ipag->transaction()->setTid('123456789')->consult();
```
### Captura

```php
$response = $ipag->transaction()->setTid('123456789')->capture();
```

### Captura Parcial

```php
$response = $ipag->transaction()->setTid('123456789')->setAmount(1.00)->capture();
```

### Cancelamento

```php
$response = $ipag->transaction()->setTid('123456789')->cancel();
```

### Cancelamento Parcial

```php
$response = $ipag->transaction()->setTid('123456789')->setAmount(1.00)->cancel();
```

## Assinatura
### Criando uma Assinatura

```php
$transaction = $ipag->transaction();

$transaction->getOrder()
    ->setOrderId($orderId)
    ->setCallbackUrl(getenv('CALLBACK_URL'))
    ->setAmount(10.00)
    ->setInstallments(1)
    ->setPayment($ipag->payment()
        ->setMethod(Method::VISA)
        ->setCreditCard($creditCard)
    )->setCustomer($customer)
)->setSubscription($ipag->subscription()
    ->setProfileId('1000000')
    ->setFrequency(1)
    ->setInterval('month')
    ->setStart('10/10/2018')
);

$response = $transaction->execute();
```

## Exemplo de Transação Completa
### Exemplo via Cartão de Crédito

```php
<?php

require 'vendor/autoload.php';

use Ipag\Ipag;
use Ipag\Classes\Authentication;
use Ipag\Classes\Endpoint;

try {
    $ipag = new Ipag(new Authentication('my_id_ipag', 'my_ipag_key'), Endpoint::SANDBOX);

    $customer = $ipag->customer()
        ->setName('Fulano da Silva')
        ->setTaxpayerId('799.993.388-01')
        ->setPhone('11', '98888-3333')
        ->setEmail('fulanodasilva@gmail.com')
        ->setAddress($ipag->address()
            ->setStreet('Rua Júlio Gonzalez')
            ->setNumber('1000')
            ->setNeighborhood('Barra Funda')
            ->setCity('São Paulo')
            ->setState('SP')
            ->setZipCode('01156-060')
    );

    $cart = $ipag->cart(
        ['Produto 1', 5.00, 1, 'ABDC1'],
        ['Produto 2', 2.50, 2, 'ABDC2']
    );

    $creditCard = $ipag->creditCard()
        ->setNumber('4066553613548107')
        ->setHolder('FULANO')
        ->setExpiryMonth('10')
        ->setExpiryYear('2025')
        ->setCvc('123');

    $transaction = $ipag->transaction();

    $transaction->getOrder()
        ->setOrderId($orderId)
        ->setCallbackUrl('https://minha_loja.com.br/ipag/callback')
        ->setAmount(10.00)
        ->setInstallments(1)
        ->setPayment($ipag->payment()
            ->setMethod(Method::VISA)
            ->setCreditCard($creditCard)
        )
        ->setCustomer($customer)
        ->setCart($cart);

    $response = $transaction->execute();

    //Retornou algum erro?
    if (!empty($response->error)) {
        throw new \Exception($response->errorMessage);
    }

    //Pagamento Aprovado (5) ou Aprovado e Capturado(8) ?
    if ($response->payment->status == '5' || $response->payment->status == '8') {
        //Faz alguma coisa...
        return $response;
    }
} catch(\Exception $e) {
    print_r($e->__toString());
}
```

## Exemplo de Transação com Regra de Split
### Exemplo via Cartão de Crédito

```php
<?php

require 'vendor/autoload.php';

use Ipag\Ipag;
use Ipag\Classes\Authentication;
use Ipag\Classes\Endpoint;

try {
    $ipag = new Ipag(new Authentication('my_id_ipag', 'my_ipag_key'), Endpoint::SANDBOX);

    $customer = $ipag->customer()
        ->setName('Fulano da Silva')
        ->setTaxpayerId('799.993.388-01')
        ->setPhone('11', '98888-3333')
        ->setEmail('fulanodasilva@gmail.com')
        ->setAddress($ipag->address()
            ->setStreet('Rua Júlio Gonzalez')
            ->setNumber('1000')
            ->setNeighborhood('Barra Funda')
            ->setCity('São Paulo')
            ->setState('SP')
            ->setZipCode('01156-060')
    );

    $cart = $ipag->cart(
        ['Produto 1', 5.00, 1, 'ABDC1'],
        ['Produto 2', 2.50, 2, 'ABDC2']
    );

    $creditCard = $ipag->creditCard()
        ->setNumber('4066553613548107')
        ->setHolder('FULANO')
        ->setExpiryMonth('10')
        ->setExpiryYear('2025')
        ->setCvc('123');

    $payment = $ipag->payment()
            ->setMethod(Method::VISA)
            ->setCreditCard($creditCard);

    //Regra de Split 1 (com porcentagem %)
    $payment->addSplitRule($ipag->splitRule()
        ->setSellerId('c66fabf44786459e81e3c65e339a4fc9')
        ->setPercentage(15)
        ->setLiable(1)
    );

    //Regra de Split 2 (com valor absoluto R$)
    $payment->addSplitRule($ipag->splitRule()
        ->setSellerId('c66fabf44786459e81e3c65e339a4fc9')
        ->setAmount(5.00)
        ->setLiable(1)
    );

    $transaction = $ipag->transaction();
    $transaction->getOrder()
        ->setOrderId($orderId)
        ->setCallbackUrl('https://minha_loja.com.br/ipag/callback')
        ->setAmount(10.00)
        ->setInstallments(1)
        ->setPayment($payment)
        ->setCustomer($customer)
        ->setCart($cart);

    $response = $transaction->execute();

    //Retornou algum erro?
    if (!empty($response->error)) {
        throw new \Exception($response->errorMessage);
    }

    //Pagamento Aprovado (5) ou Aprovado e Capturado(8) ?
    if ($response->payment->status == '5' || $response->payment->status == '8') {
        //Faz alguma coisa...
        return $response;
    }
} catch(\Exception $e) {
    print_r($e->__toString());
}
```

## Exemplo de Página de Callback

```php
<?php

require_once 'vendor/autoload.php';

use Ipag\Classes\Services\CallbackService;

$postContent = file_get_contents('php://input');

$callbackService = new CallbackService();

// $response conterá os dados de retorno do iPag
// $postContent deverá conter o XML enviado pelo iPag
$response = $callbackService->getResponse($postContent);

// Verificar se o retorno tem erro
if (!empty($response->error)) {
    echo "Contem erro! {$response->error} - {$response->errorMessage}";
}

// Verificar se a transação foi aprovada e capturada:
if ($response->payment->status == '8') {
    echo 'Transação Aprovada e Capturada';
    // Atualize minha base de dados ...
}
```

## Resposta

Estrutura do Transaction Response:

- transaction
    - id
    - tid
    - authId
    - amount
    - acquirer
    - acquirerMessage
    - urlAuthentication
    - urlCallback
    - createdAt
    - creditCard
        - holder
        - number
        - expiry
        - brand
        - token
    - subscription
        - id
        - profileId
    - payment
        - status
        - message
    - order
        - orderId
    - customer
        - name
        - email
        - phone
        - cpfCnpj
        - address
            - street
            - number
            - district
            - complement
            - city
            - state
            - zipCode
    - antifraud
        - id
        - score
        - status
        - message
    - splitRules
        - [0]
            - rule
            - seller_id
            - ipag_id
            - amount
            - amount
            - percentage
            - liable
            - charge_processing_fee
    - error
    - errorMessage
    - history
        - [0]
            - amount
            - operationType
            - status
            - responseCode
            - responseMessage
            - authorizationCode
            - authorizationId
            - authorizationNsu
            - createdAt

## Testes

Os Tests Unitários são realizados contra o Sandbox do iPag, o arquivo de configuração (phpunit.xml) já vem preenchido com um acesso limitado ao Sandbox.

É necessário a instalação do PHPUnit para a realização dos testes.

## Licença
[The MIT License](https://github.com/jhernandes/ipag-sdk-php/blob/master/LICENSE)

## Documentação

[Documentação Oficial](https://docs.ipag.com.br)

## Dúvidas & Sugestões

Em caso de dúvida ou sugestão para o SDK abra uma nova [Issue](https://github.com/jhernandes/ipag-sdk-php/issues).
