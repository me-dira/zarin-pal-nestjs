# Zarinpal Adaptor for `NestJs`.

Use zarinpal payment features as easy as blink on Nestjs framework.
Do not use this package for your Business packages due to this package is under development and it's not tested properly but stay `tunned` because we will release stable version in this weekend.

# Features:
- Easy to use.
- Light and fast.
- Any dependency to other zarinpal packages.
- Specific error type (ZarinpalError).

## Version 1 goals
- Verify section tests.
- Better error typing.
- Currency option.
- More Tests ...

## How to use:
The process is very simple and includes:

- Open transaction (on Zarinpal server) and get result.
- Redirect user to generated URL.
- Verify transaction. If not Zarinpal will return the price to user.

---

### installation
Install is very simple
Before anything you should get and install package from `npm`:

```
yarn add zarinpal-nestjs
// Or
npm install --save zarinpal-nestjs
```

### Module registration
After successful installation you need to register zarinpal-nestjs module in your NestJs application:

```
@Module({
  imports: [
    ZarinpalModule.register({
      callBackUrl: 'https://google.com',
      merchantId: '32 character long merchant_id',
    }),
  ],
})
export class AppModule {}

```

---

### Open transaction on `Zarinpal` servers.
Lets `inject` and use our package inside controller
but you need to store income opened transaction because you need it after user is returned to verify endpoints. Store it and search it on verify method.

Below code is just a sample and you can use and inject services everywhere you like.

```
import {
  ZarinpalError,
  ZarinpalService,
  ZarinpalVerifyQueryParams,
} from 'zarinpal-nestjs';


// Inside controller

@Controller('transaction')
export class AppController {
  constructor(
    private readonly zarinpalService: ZarinpalService,
    private transactionService: TransactionService,
  ) {}

  /**
  * This endpoint opens transaction and generates
  * start pay url and returns it. You need to redirect
  * user to this url. after user done the payment process
  * i will be redirected to url you registered on your Main module.
   */
  @Get('open')
  async openTransaction(): Promise<string> {
    try {
      const transactionResult = await this.zarinpalService.openTransaction({
        amount: 1000,
        description: 'Buying a car (example)',
      });

    // In real word, you need to store the result
    // Generate start pay url

    return this.zarinpalService.generateStartPayUrl(transactionResult);

    } catch (e) {
      if (e instanceof ZarinpalError) {
        throw new HttpException(e.message, e.status);
      }
      throw e;
    }
  }

  @Get('verify')
  async verifyTransaction(@Query() queryParams: ZarinpalVerifyQueryParams) {
    const transaction = this.transactionService.findByAuthority(
      queryParams.Authority,
    );

    if (queryParams.Status === 'OK') {
      return await this.zarinpalService.verifyRequest({
        authority: queryParams.Authority,
        amount: transaction.amount,
      });
    }

    else {
      throw new BadRequestException('پرداخت به دست کاربرلغو شده است');
    }
  }
}

```
