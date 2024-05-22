# Set up Drop-in

## Get powerboard commercetools

### npm or yarn (recommended)

```
    npm install @powerboard-commercetools/powerboard
```

```
    yarn add @powerboard-commercetools/powerboard
```

Import powerboard into your application. You can add your own styling by overriding the rules in the CSS file.

```
    import PowerboardCommercetoolWidget from import('@powerboard-commercetools/powerboard');
    import '@powerboard-commercetools/powerboard/dist/widget.css';
```


### Embed script and stylesheet

Embed the Powerboard Commercetools script element above any other JavaScript in your checkout page.

```
    <script src="powerboard-commercetools/widget.js"></script>
```

Embed the Powerboard Commercetools stylesheet. You can add your own styling by overriding the rules in the CSS fil

```
    <link rel="stylesheet" href="powerboard-commercetools/widget.css">
```


## Create a DOM element for Drop-in

Create a DOM container element on your checkout page where you want Drop-in to be rendered and give it a descriptive id. 

```
    <div id="powerboard-widget-container">
        <!-- Powerboard Checkout will be mounted here -->
    </div>
```

## Create store for Drop-in

Сreate a global store where the properties of each payment method will be written when the widget is initialized.

```
    import { reactive } from 'vue';

    const powerboardStore = reactive({});

    export default powerboardStore;
```

## Initialize the payment session. Example on Vue.js

Create an instance of Drop-in and mount it to the container element you created.

### 1. Load powerboard script

Add function load powerboard script

```
    const PRODUCTION_POWERBOARD_URL = 'https://widget.powerboard.commbank.com.au/sdk/latest/widget.umd.js';
    const SANDBOX_POWERBOARD_URL = 'https://widget.preproduction.powerboard.commbank.com.au/sdk/latest/widget.umd.js';
```


### 2. Get powerboard payment configuration 

First of all you need to get information about the cart, customer, and methods for working with the cart

Then add function get payment configuration. Set Powerboard payment configurations paymentId, paymentVersion, api_commercetools.

```
    const getPowerboardPaymentsConfiguration = async () => {

        // Fetch payment methods using a POST request       
        let response = await fetchWithToken(`${config.ct.api}/${config.ct.auth.projectKey}/payments/`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({
                amountPlanned: {
                    currencyCode: currencyCode,
                    centAmount: totalPrice
                },
                paymentMethodInfo: {
                    paymentInterface: 'Mock',
                    method: 'powerboard-pay',
                    name: {
                        en: 'Powerboard'
                    }
                },
                custom: {
                    type: {
                        typeId: 'type',
                        key: "powerboard-components-payment-type"
                    },
                    fields: {
                        commercetoolsProjectKey: config.ct.auth.projectKey,
                        PaymentExtensionRequest: JSON.stringify({action:"getPaymentMethodsRequest",request: getPaymentMethodsRequest})
                    }
                },
                transactions: [
                    {
                        type: "Charge",
                        amount: {
                            currencyCode: "AUD",
                            centAmount: totalPrice
                        },
                        state: "Initial"
                    }
                ]
            }),
        });
    }
```

### 3. Add function initialize powerboard checkout.

Function is responsible for

- Get the parameters
- Creating a widget
- Display of payment methods
- Widget storage and event handling

Handle specific logic for saving card details. Сheck if the user is logged and widget.isSaveCardEnable() equal true, use widget render methods.

```
    widget.renderSaveCardCheckbox(); 
    widget.renderCredentialsSelect();

```

Set amount and currency for the widget based on the cart data

```
    widget.setAmount(totalPrice);
    widget.setCurrency(currencyCode);
```

Display payment methods on the widget

```
    widget.displayPaymentMethods(paymentMethod);
``` 

Load the widget (card, bank)

```
    widget.loadWidget()
```

Get widget

```
    widget.widget
```

Create a new widget instance

```
    async function initPowerboardCheckout(paymentMethod, powerboardStore, configuration, PowerboardCommercetoolWidget) {
        let widget = new PowerboardCommercetoolWidget({
            selector: '#' + paymentMethod.name,
            type: paymentMethod.type,
            configuration: configuration,
            userId: commerceToolCustomerId,
            paymentButtonSelector: '#paymentButton',
            radioGroupName: 'payment_method',
        });
    }
```

### 4. Component mount 

```
    onMounted(async () => {

        Object.values(configuration.payment_methods).forEach(paymentMethod => {
            Object.entries(paymentMethod.config).forEach(([key, value]) => {
                if (key.includes('use_on_checkout') && value === 'Yes') {
                    initPowerboardCheckout(paymentMethod, powerboardStore, configuration, PowerboardCommercetoolWidget);
                }
            });
        });

    })
```

### 5. Add a custom hook for handling Powerboard payments

**During order placement, if the checkout form is valid and the payment method is powerboard, we will provide the function of creating an order through powerboard in which:**

1. We receive the value of a one-time OTT token from the widget

```
    input[name="payment_source_card_token"]
```

2. We collect the required data and transfer it to the widget

```
    widget.setAmount(totalPrice)
    widget.setCurrency(currencyCode)
    widget.currencyCode.setPaymentSource(paymentSource)
    widget.setAdditionalInfo(additionalInfo)
```

3. Get vault token for the payment

```
    widget.getVaultToken()
```

4. We create a payment by updating the existing commercetools api "makePaymentRequest"

Create payment using the collected data

```
    createPayment({...}) 
```

5. Link the payment to the order and create the order


### 6. Transfer information to widget

To pass payment information, pass the objects of this structure to the setBillingInfo() and setShippingInfo() methods.

```
    setBillingInfo({
                first_name: "first_name", //string
                last_name: "last_name"l, //string
                email: "email", //string
                phone: "phone"l, //string
                address_line1: "address_line1", //string
                address_line2: "address_line2", //string
                address_city: "address_city", //string
                address_state: "address_state", //string
                address_country: "address_country", //string
                address_postcode: "address_postcode", //integer
    });
```

for cart item information pass the object of this structure to the setCartItems method.

```
    setCartItems([
    {
            name: "name of product", // string
            type: "type", // string  (type or category name of product)
            quantity: 10, // int
            item_uri: "https://some.domain/path/to/product", // string
            image_uri: "https://cdn.some.domain/path/to/product/image",
            amount: 123.45 // float (price with two digits after the decimal point)
    }
    ]);
```


### 7. Thank You page

After making the payment, it redirect to the Thank you page and the text is displayed depending on the completion of the payment.

If status pending 'yes', we use text

```
    "thankYouOrderProcessed": "Your order is being processed. We’ll get back to you shortly."
 ```

If status pending 'no', we use text

```
    "thankYouOrderReceived": "Thank you. Your order has been received.",
 ```

Use the function redirectToThankYouPage to receive a text status about the order.

```
     async function redirectToThankYouPage(router) {
        let currentPaymentUrl = `${config.ct.api}/${config.ct.auth.projectKey}/payments/${powerboardStore?.paymentId}`;
        const response = await fetchWithToken(currentPaymentUrl, {
            method: 'GET',
            headers: {'Content-Type': 'application/json'}
        });

        let orderStatusInReview = currentPayment.custom.fields.PowerboardPaymentStatus === 'powerboard-pending' ? 'yes' : 'no'
    }
```