# Pix Credit - Request For Comment (RFC)

This document proposes an approach on implementing credit transactions based on pix.

## Motivation

Pix can be a building-block for more complex implementations. Pix API is open-sourced by Banco Central do Brasil (BACEN). This allows companies to use it for developing new products on top of it. This RFC proposes Pix Credit, credit built on top of pix. 

## Definitions

### What's pix?

Pix[^1] is an inovative instant payment method created and patented by BACEN. As far as noticed no other country have a similar implementation. All it takes to work is someone with an active bank account and internet. 

#### Basic Working Flow

Pix itself only transfers money from account A to account B (in seconds). Besides other features is has, that's all it does in summary. 

Pix transactions have no fees. For instance when person A pays a charge from person B, none of them pay any fees. Person B pays no fee for generating charges and person A pays no fee during payment. 

### What's credit?

Credit[^2] is a money borrowing mechanism where people buy and pay later. The most known method for credit access is a credit card. All it takes to work is a bank account and credit available for use. 

#### Basic Working Flow 

Every transaction uses available debit from the credit balance. For instance customer A can spend BRL 1.500 (one thousand fivehundred reais) when BRL 3.000 (three thousand reais) is available as credit. And fees are charged from customer A if the credit card bill becomes past due. 

## Architecture

[ ] - Add draw here where bacen's api is connected to woovi's api  and customer/company sends charges to it (type of pix credit/product is not specified yet)

## Pix Credit

[ ] - Explain what is pix credit (maybe add an image here as well)

[ ] - Explain how credit is approved/rejected (maybe add an image here as well)

[ ] - Explain how we make money from transactions (maybe add an image here as well)

[ ] - Explain pix credit risks for bank and customers (maybe add an image here as well)

### How we make money

### Basic Work Flow

[ ] - Explain how a charge is generated, how fees are calculated and when they're charged (maybe add an image here as well)

### Use Cases

[ ] - Initially create one use case (maybe add an image here as well)

[^1]:https://www.bcb.gov.br/en/financialstability/pix_en
[^2]:https://www.experian.com/blogs/ask-experian/credit-education/faqs/what-is-credit/