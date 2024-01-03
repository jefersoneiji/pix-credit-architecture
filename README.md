# Pix Credit - Request For Comment (RFC)

This document proposes an approach for implementing credit transactions based on pix.

## Motivation

Pix can be a building block for more complex implementations. Pix API is open-sourced by Banco Central do Brasil (BACEN). This allows companies to use it for developing new products on top of it. This RFC proposes Pix Credit, credit built on top of Pix. 

## Definitions

### What's pix?

Pix[^1] is an innovative instant payment method created and patented by BACEN. As far as noticed no other country has a similar implementation. All it takes to work is someone with an active bank account and internet access. 

#### Basic Working Flow

Pix itself only transfers money from account A to account B (in seconds). Besides other features it has, that's all it does in summary. 

Pix transactions have no fees. For instance, when person A pays a charge from person B, none of them pay any fees. Person B pays no fee for generating charges and person A pays no fee during payment. 

### What's credit?

Credit[^2] is a money-borrowing mechanism where people buy and pay later. The most known method for credit access is a credit card. All it takes to work is a bank account and credit available for use. 

#### Basic Working Flow 

Every transaction uses available debit from the credit balance. For instance, customer A can spend BRL 1.500 (one thousand five hundred reais) when BRL 3.000 (three thousand reais) is available as credit. Fees are charged from customer A if the credit card bill becomes past due. 

## Architecture

### Pix 

Pix is handled by BACEN infrastructure through "Sistema de Pagamentos Instantâneos (SPI)"[^3]. And I quote: 

>O Sistema de Pagamentos Instantâneos (SPI) é a infraestrutura centralizada e única para liquidação de pagamentos instantâneos entre instituições distintas no Brasil.
>
>O SPI é um sistema que faz liquidação bruta em tempo real (LBTR), ou seja, que processa e liquida transação por transação. Uma vez liquidadas, as transações são irrevogáveis.
>
>Os pagamentos instantâneos são liquidados com lançamentos nas contas de propósito específico que as instituições participantes diretos do sistema mantêm no BCB, denominadas Contas Pagamento Instantâneo (Contas PI). Para garantir a solidez do sistema, não há possibilidade de lançamentos a descoberto, isso é, não se admite saldo negativo nas Contas PI.

#### Accessing SPI

![Accessing SPI](./assets/pix-architecture-bacen.png "accessing spi")

Image source: BACEN[^4]
 
### Credit Card

Credit card payment processing is defined as a "Payment Arrangement" meaning that several players participate in this process. As a consequence, many different infrastructures are chained up to the point where merchants are paid and customers get their final bill.[^5] I quote: 

>Um arranjo de pagamento é um conjunto de regras e procedimentos que disciplina a prestação de determinado serviço de pagamento ao público. Diferentemente da compra com dinheiro vivo entre duas pessoas que se conhecem, o arranjo conecta todas as pessoas que a ele aderem. É o que acontece quando o cliente usa uma bandeira de cartão de crédito numa compra que só é possível porque o vendedor aceita receber daquela bandeira.

#### Accessing Credit
O fluxo em azul representa o que chamamos de autorização, processo em que o emissor aprova ou rejeita uma transação. Em verde temos o processo de liquidação desta transação.

The blue line represents the "authorization flow", in this process, a transaction is either rejected or approved. 

The green line represents the "settlement flow", where the **Merchant** collects his money from the sale. 

![Accessing SPI](./assets/credit-card-architecture.webp "accessing credit")

Image source: Lucas Rosa[^5]

**<p align="center">Subtitle</p>**

| Role       | Function                                                                                                         |
|------------|------------------------------------------------------------------------------------------------------------------|
| Customer   | Is a cardholder, someone who owns a credit card.                                                               |
| Merchant   | Store who sells a service or product.                                                                            |
| Acquirer   | Institution that bridges merchants and card brands. Providers of credit card machines used in POS for instance. Just to mention a few Cielo, Stone and SumUp.                                                             |
| Card Brand | They're responsible for bridging customers, acquirers and issuers. The most famous ones are Visa and Mastercard. |
| Issuer     | Financial institution responsible for paying acquirers and collecting money from customers.                      |
| POS        | Point Of Sale, a device that merchants use to charge their clients.                                                |

## Pix Credit

Pix Credit combines the agility of payment from Pix with the easiness of borrowing money through credit. 

### Basic Working Flow

Let's imagine a Marchant adopted Pix Credit (PC) in their Point Of Sale (POS). Thus as expected a customer makes a purchase using the PC payment option. 

1. A Pix charge is generated in Woovi's platform from the **Merchant** side
2. The **Customer** opens his/her Pix Credit Section provided by the **Issuer**
3. This Pix Charge is paid by the **Customer**
4. The amount for this purchase is charged from Pix Credit **Customer** balance
5. **Merchant** cashes in sales amount
6. For the next purchase, all steps above may be repeated until no balance is left in **Customer**'s Pix Credit balance 

## Business Rules

Next, all the steps above are dissected in the context of business rules. 

### 1. Charge Generation

In essence there are **three sources of revenue**[^6] for Pix Credit: 

1. Interest from Monthly bill (**Customer**)
2. Fees from past due bills or cash advance (**Customer**)
3. Interchange also known as transaction fees (**Merchant**)

Considering the charge generation mentioned before, one fee is applied **(Interchange)**:  The **Merchant** pays up to **8%** of the total sold. For a sale of **$3.000,00 (three thousand)** up to **$240** is charged from him/her. 

### 2. Customer Pix Credit Access

In this step, the **Customer** needs a means to pay his debt. So his/her banking app transactions the available balance. There are two possible situations: 

1. Customer **has balance available** so this money is debited
2. Customer **uses Pix Credit balance** thus borrowing money from his **Issuer**. Later on, this pending payment becomes a Pix Credit Bill

Now, back to the basic working flow...this money is withdrawn from option 1.

### 3. Money is Withdraw from the Customer's Account

Because the **Customer** already had money in his/her account money was withdrawn from it. **Therefore, no Pix Credit Bill caused by borrowing was generated**.

### 4. The customer's balance is debited

Considering the previous step, that transaction works almost the same as a debit card. 

When money is withdrawn from Pix credit through borrowing, the **Issuer** is responsible for taking the borrowing risk. Moreso, the **Issuer** also acts in the settlement phase where **Merchants** get paid after this transaction is authorized by the **Issuer** itself. 

### 5. Merchant cashes in money from the sale

For sales where the **Customer** used his/her account balance, cash-in is almost instantaneous for **Merchant**. Pix cash-in **time is a max of 10 seconds**. 

On the other hand, for sales where the **Customer** used his/her Pix Credit through borrowing time for funding may vary between 1 and 3 business days[^7]. 

### 6. Balance debiting from customer

As the last step in this basic working flow the balance was debited similarly to a debit purchase. Therefore this process would repeat itself to the point where no balance is left in the account for future **Customer** purchases. 

Now considering that this purchase used Pix Credit balance, new purchases would use the credit balance, up to the point where the credit balance hits $0 (zero) available for the **Customer**.

**What would happen when the credit balance goes $0 (zero)?** The **Issuer** rejects new purchases from this **Customer**.

**Is it possible for a **Customer** to make new purchases even without credit available?** Yes. In this case, the **Issuer** steps in and may provide a new limit or new loan for the **Customer**. All of this requires **Risk Analysis**. 

## Risks

This section explains the risks in Pix Credit Transactions followed by ways to avoid them while maintaining profitability.

### For the Customer 

There are two worth mentioning here: 

1. Missing Pix Credit bills therefore being charged with fees for the next bill
**Possible solution**: **Customer** score analysis before approving credit limits or any type of upgrading

2. Credit fraud, someone with bad intentions taking over the **Customer** sensitive data. Thus making unauthorized purchases <br>
**Possible solution**: Two-factor authentication in **Customer** access to **Issuer** services

### For the Issuer

Two main risks will be mentioned next: 

1. Yet, credit card fraud <br>
**Possible solutions**: Improvements in Payment Fraud Detection[^8] 

2. **Customer** misses paying credit bills <br>
**Possible solutions**: 
    - **Issuer** may offer Revolving Credit conditions[^9]. Revolving Credit means that a lower credit amount is available when the **Customer** didn't pay the full bill but partially
    - **Issuer** may securitize[^10] their credit card receivables meaning that all pending receivables are sold to the market

[^1]:https://www.bcb.gov.br/en/financialstability/pix_en
[^2]:https://www.experian.com/blogs/ask-experian/credit-education/faqs/what-is-credit/
[^3]:https://www.bcb.gov.br/estabilidadefinanceira/sistemapagamentosinstantaneos
[^4]:https://www.bcb.gov.br/content/estabilidadefinanceira/SPI_IMG/info_sistema_pagamentos_instantaneos_acesso%20ao%20SPI_br_0170_2020.png
[^5]:https://lucascmrosa.medium.com/sistemas-de-pagamentos-i-cart%C3%B5es-53ece499f9e3
[^6]:https://www.nerdwallet.com/article/credit-cards/credit-card-companies-money
[^7]:https://www.clearlypayments.com/blog/how-long-does-it-take-to-get-funds-in-credit-card-processing/#:~:text=In%20general%2C%20credit%20card%20processing,with%20a%20merchant%20account%20provider.
[^8]:https://www.mastercardservices.com/en/portfolio-intelligence-solutionswith%20a%20merchant%20account%20provider.
[^9]:https://www.investopedia.com/terms/r/revolvingcredit.aspwith%20a%20merchant%20account%20provider.
[^10]:https://crdc.com.br/o-que-e-securitizacao-de-recebiveis-e-como-funciona-um-guia-completo