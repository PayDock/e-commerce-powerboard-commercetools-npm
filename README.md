# PowerBoard Commercetools

## Table of Contents

* [Install PowerBoard Commercetools](#install-powerboard-commercetools)
* [Embed your Script and Stylesheet](#embed-your-script-and-stylesheet)
* [Create a DOM element for Drop-in](#create-a-dom-element-for-drop-in)
* [Setup Drop-in](#setup-drop-in)
* [Initialize the Payment Session](#initialize-the-payment-session)

Create an end-to-end checkout experience for your shoppers using PowerBoard's Commercetools.

## Install PowerBoard Commercetools

1. Install the [PowerBoard Commercetools Node package](https://www.npmjs.com/package/@power-board-commercetools/powerboard)

2. Install the PowerBoard Commercetools using either npm or yarn. The two commands for this are as follows:

```bash
npm install @power-board-commercetools/powerboard
```

```bash
yarn add @power-board-commercetools/powerboard
```

3. Import PowerBoard into your application. Add your own styling by overriding the rules in the CSS file.

```javascript
import PowerboardCommercetoolWidget from '@power-board-commercetools/powerboard';
import '@power-board-commercetools/powerboard/dist/widget.css';
```


### Embed script and stylesheet

1. Embed the PowerBoard Commercetools script element a the beginning of your JavaScript file in your Checkout page.

```html
<script src="powerboard-commercetools/widget.js"></script>
```

2. Embed the PowerBoard Commercetools stylesheet. You can add your own styling by overriding the rules in the CSS file.

```html
<link rel="stylesheet" href="powerboard-commercetools/widget.css">
```


## Create a DOM element for Drop-in

1. Create a DOM container element on your checkout page.

```html
<div id="powerboard-widget-container">
     <!-- PowerBoard Checkout will be mounted here -->
</div>
```

2. Place the DOM container element where you want Drop-in to be rendered, and provide a descriptive ID for your div element.

## Set up Drop-in

1. Сreate a global store for Drop-in. 

2. When the widget is initialized, the properties of each payment method are written to the global store. 

```javascript
import { reactive } from 'vue';

const powerboardStore = reactive({});

export default powerboardStore;
```

## Initialize the payment session

The following example demonstrates how to initialize a payment session with Vue.js.

To do this you must create an instance of Drop-in and then mount the instance to the container element created in the setup section. Detailed instructions describing how to create and mount Drop-in are as follows:

### 1. Load the PowerBoard script

Load the Paydock script. Ensure that all logic related to the widget and widget initialization occurs after the file is loaded. For example:

```javascript
import {loadScript} from "vue-plugin-load-script";

const PRODUCTION_POWERBOARD_URL = 'https://widget.powerboard.commbank.com.au/sdk/latest/widget.umd.js';
const SANDBOX_POWERBOARD_URL = 'https://widget.preproduction.powerboard.commbank.com.au/sdk/latest/widget.umd.js';
const configuration = await getPowerboardPaymentsConfiguration();
const isSandbox = configuration?.sandbox_mode; 

await loadScript(isSandbox === 'Yes' ? SANDBOX_POWERBOARD_URL : PRODUCTION_POWERBOARD_URL)
```
### 2. Set configuration data

The following is an example of the setup for your config data. 

```javascript
const config = {
  api: 'https://api.europe-west1.gcp.commercetools.com',
  auth: {
    host: 'https://auth.europe-west1.gcp.commercetools.com',
    projectKey: 'powerboard',
    credentials: {
      clientId: 'some-client-id',
      clientSecret: 'some-client-secret',
      scope: 'all_neaded scopes for work yor store example "manage_orders:powerboard manage_customers:powerboard"'
    },
  }
}
```

### 3. Get PowerBoard payment configuration

To initialize the checkout, you need the configuration information about the cart, customer, and the methods for working with the cart.

The following response demonstrates the full configuration, the available payment methods, and the unique payment ID.

```javascript
import axios from "axios";

const getPaydockPaymentsConfiguration = async () => {
    // Fetch payment methods using a POST request       
    let response = await axios.post(`${config.api}/${config.auth.projectKey}/payments/`, {
        amountPlanned: {
            currencyCode: 'AUD',
            centAmount: 12415 //integer, amount in cents
        },
        paymentMethodInfo: {
            paymentInterface: 'Mock',
            method: 'powerboard-pay',
            name: {
                en: 'PowerBoard'
            }
        },
        custom: {
            type: {
                typeId: 'type',
                key: "powerboard-components-payment-type"
            },
            fields: {
                commercetoolsProjectKey: config.auth.projectKey,
                PaymentExtensionRequest: JSON.stringify({
                    action: "getPaymentMethodsRequest",
                    request: {}
                })
            }
        },
        transactions: [
            {
                type: "Charge",
                amount: {
                    currencyCode: "AUD",
                    centAmount: 12415 //integer, amount in cents
                },
                state: "Initial"
            }
        ]
    }, {
        headers: {
            'Content-Type': 'application/json',
            authorization: `Bearer ${await this.getAuthToken()}`
        }
    });
}
```

### 4. Add a function that initializes the PowerBoard checkout

The following function initializes the PowerBoard checkout. You can use this function to:

- Get the parameters
- Create a widget
- Display payment methods
- Use widget storage and event handling


1. Create a new widget.

```javascript
function initPowerboardCheckout(paymentMethod, paydockStore, configuration, PowerboardCommercetoolWidget) {
    configuration.api_commercetools = {
        url: `${config.ct.api}/${config.ct.auth.projectKey}/payments/`,
        token: localStorage.getItem(ACCESS_TOKEN)
    }
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

2. Handle the logic for saving card details. Check that the customer is logged in and that widget.isSaveCardEnable() equals true. Use the widget render methods.


```javascript
widget.renderSaveCardCheckbox(); 
widget.renderCredentialsSelect();

```

3. Set the amount and currency for the widget based on the cart data.

```javascript
widget.setAmount(totalPrice);
widget.setCurrency(currencyCode);
```

4. Display the payment methods on the widget.

```javascript
widget.displayPaymentMethods(paymentMethod);
``` 

5. Load the widget.

```javascript
widget.loadWidget()
```

6. Get the widget.

```javascript
widget.widget
```

### 5. Mount the Component

```javascript
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

### 6. Add a custom hook for handling PowerBoard payments

**Note:** During order placement, if the checkout form is valid and the payment method is PowerBoard, we provide the function of creating an order through PowerBoard where:

1. PowerBoard receives the value of a one-time OTT token from the widget.

```javascript
input[name="payment_source_card_token"]
```

2. PowerBoard collects the required data and transfers it to the widget.

```javascript
widget.setAmount(totalPrice)
widget.setCurrency(currencyCode)
widget.currencyCode.setPaymentSource(paymentSource)
widget.setAdditionalInfo(additionalInfo)
```
3. For wallets you must set a validation state in the form:

```javascript
widget.setIsValidForm(true);
```

3. PowerBoard gets the vault token for the payment.

```javascript
widget.getVaultToken()
```

4. PowerBoard creates a payment by updating the existing Commercetools API "makePaymentRequest".

### 7. Create a payment using the collected data

Create a payment with the collected data using the following function:

```javascript
createPayment({...}) 
```

You must perform 3 actions in the createPayment function:

1. First you must create the payment.  

Send a request to create a payment in the PowerBoard system through the extension. Use the
custom field makePaymentRequest where "widget.paymentId" is the unique payment ID from step 3, [Get PowerBoard Payment configuration](#3-get-powerboard-payment-configuration).

```javascript
//example for "Google Pay"

let powerboardResponse = document.querySelector('[name="powerboard-pay-google-pay"]').value;
let chargeId = powerboardResponse.data.id;
let paymentType = 'Google Pay';
let currentPaymentUrl = `${config.ct.api}/${config.ct.auth.projectKey}/payments/${widget.paymentId}`;
 
powerboardResponse = JSON.parse(powerboardResponse);
 
if (powerboardResponse.data.status === "inreview") {
    status = 'powerboard-pending'
} else {
    status = powerboardResponse.data.status === 'pending' ? 'powerboard-authorize' : 'powerboard-paid';
}
 
let response = await fetchWithToken(currentPaymentUrl, {
    method: 'GET',
    headers: headers
});
let currentPayment = await response.json();
        
const updateData = {
    version: currentPayment.version,
    actions: [
        {
             action: "setCustomField",
             name: "makePaymentRequest",
             value: JSON.stringify({
                 orderId: widget.paymentId,
                 paymentId: widget.paymentId,
                 amount: {
                     currency: currencyCode,
                     value: centAmount
                 },
                 PowerboardTransactionId: paymentSource,
                 PowerboardPaymentStatus: status,
                 PowerboardPaymentType: paymentType,
                 CommerceToolsUserId: commerceToolCustomerId,
                 SaveCard: saveCard,
                 VaultToken: vaultToken,
                 AdditionalInfo: additionalInfo
             })
        }
   ]
};
const response = await fetchWithToken(currentPaymentUrl, {
    method: 'POST',
    headers: headers,
    body: JSON.stringify(updateData)
});
let payment = await response.json();
let powerboardWidgetCardServerErrorBlock = document.getElementById('powerboard-widget-card-server-error');
let paymentExtensionResponse = payment?.custom?.fields?.PaymentExtensionResponse ?? null
if (paymentExtensionResponse) {
    paymentExtensionResponse = JSON.parse(paymentExtensionResponse);
    if (paymentExtensionResponse.status === "Failure") {
        powerboardWidgetCardServerErrorBlock.innerText = paymentExtensionResponse.message;
        powerboardWidgetCardServerErrorBlock.classList.remove("hide");
        return Promise.reject(paymentExtensionResponse.message);
    }
}

```

2. Add the payment to the cart.

```javascript
response = await fetchWithToken(`${config.ct.api}/${config.ct.auth.projectKey}/carts/${cartId}`, {
    method: 'GET',
    headers: headers
});
let currentCart = await response.json();
response = await fetchWithToken(`${config.ct.api}/${config.ct.auth.projectKey}/carts/${cartId}`, {
    method: 'POST',
    headers: headers,
    body: JSON.stringify({
        version: currentCart.version,
        actions: [
            {
                action: "addPayment",
                payment: {
                    typeId: "payment",
                    id: payment.id
                }
            }
        ]
    }),
});
```
3. Process the order.

```javascript
currentCart = await response.json();
await fetchWithToken(`${config.ct.api}/${config.ct.auth.projectKey}/orders`, {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify({
        id: currentCart.id,
        orderNumber: reference,
        version: currentCart.version
    }),
});
```

4. Link the payment to the order and create the order.


### 8. Transfer information to the widget

1. To pass payment information, pass the objects of this structure to the setBillingInfo() and setShippingInfo() methods.

```javascript
setBillingInfo({
    first_name: "first_name", //string
    last_name: "last_name", //string
    email: "email", //string
    phone: "phone", //string
    address_line1: "address_line1", //string
    address_line2: "address_line2", //string
    address_city: "address_city", //string
    address_state: "address_state", //string
    address_country: "address_country", //string
    address_postcode: "address_postcode", //integer
});
```

2. For cart item information, pass the object of this structure to the setCartItems method.

```javascript
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

### 8. Thank You page

After making the payment, your customer is redirected to the Thank you page. Text is then displayed depending on the completion status of the payment.

If the status is pending 'yes', the following text displays:

```json
{
    "thankYouOrderProcessed": "Your order is being processed. We’ll get back to you shortly."
}
 ```

If the status is pending 'no', the following text displays:

```json
{
  "thankYouOrderReceived": "Thank you. Your order has been received."
}
 ```

Use the function redirectToThankYouPage to receive a text about the status of the order.

```javascript
async function redirectToThankYouPage(router) {
    let currentPaymentUrl = `${config.ct.api}/${config.ct.auth.projectKey}/payments/${powerboardStore?.paymentId}`;
    const response = await fetchWithToken(currentPaymentUrl, {
        method: 'GET',
        headers: {'Content-Type': 'application/json'}
    });

    let orderStatusInReview = currentPayment.custom.fields.PowerboardPaymentStatus === 'powerboard-pending' ? 'yes' : 'no'
}
```
## See also

- [PowerBoard website](https://www.commbank.com.au/business/payments/take-online-payments/powerboard.html)

## License

This repository is available under the [MIT license](LICENSE).
