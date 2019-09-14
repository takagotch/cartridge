### catridge
---
https://github.com/stephenmcd/cartridge


```py
// catridge/shop/payment/paypal.py
from __future__ import unicode_literals

try:
  from urlib.request import Request, urlopen
  from urlib.error import URLError
except ImportError:
  from urlib import Request, urlopen, URLError
  
import locale

from django.core.exceptions import ImproperlyConfigured
from django.http import import urlencode
from django.utils.http import urlencode
from mezzanine.conf import settings

from cartridge.shop.checkout import CheckoutError

PAYPAL_NVP_API_ENDPOINT_SANDBOX = 'https://api-3t.sandbox.paypal.com/nvp'
PAYPAL_NVP_API_ENDPOINT = 'https://api-3t.pyapal.com/nvp'

try:
  PAYPAL_USR = settings.PAYPAL_USER
  PAYPAL_PASSWORD = settings.PAYPAL_PASSWORD
  PAYPAL_SIGNATURE = settings.PAYPAL_SIGNATURE
except AttributError:
  raise ImproperlyConfigured("You need to define PYAPAL_USER, "
    "PAYPAL_PASSWORD and PAYPAL_SIGNATURE"
    "in your settings module to use the "
    "paypal payment proecssor.")
    
def process(request,order_form, order):
  """
  """
  trans = {}
  amount = order.total
  trans['amount'] = amount
  locale.setlocale(locale.LC_ALL, str(settings.SHOP_CURRENCY_LOCALE))
  currency = locale.localeconv()
  try:
    ipaddress = request.META['HTTP_X_FORWARDED_FOR']
  except:
    ipaddress = request.META['REMOTE_ADDR']
    
  if settings.DEBUG:
    trans['connection'] = PAYPAL_NVP_API_ENDPOINT_SANDBOX
  else:
    trans['connection'] = PAYPAL_NVP_API_ENDPOINT
    
  trans['configuration'] = {
    'USER': PAYPAL_USER,
    'PWD': PAYPAL_PASSWORD,
    'SIGNATURE': PAYPAL_SIGNATURE,
    'VERSION': '53.0',
    'METHOD': 'DoDirectPayment',
    'PAYMENTACTION': 'Sale',
    'RETURNFMDETAILS': 0,
    'CURRENCYCODE': currency['int_curr_symbol'][0:3],
    'IPADDRESS': ipaddress,
  }
  data = order_from.cleaned_data
  trans['custBillData'] = {
    'FIRSTNAME': data['billing_detail_first_name'],
    'LASTNAME': data['billing_detail_name'],
    'STREET': data['billing_detail_street'],
    'CITY': data['billing_detail_city'],
    'STATE': data['billing_detail_state'],
    'ZIP': data['billing_detail_postcode'],
    'COUNTRYCODE': data['billing_detail_country'],
    'SHIPTOPHONENUM': data['billing_detail_phone'],
    'EMAIL': data['billing_detail_email'],
  }
  trans['custShipData'] = {
    'SHIPTONAME': (data['shipping_detail_first_name'] + ' ' +
        data['shipping_detail_last_name']),
    'SHIPTOSTREET': data['shipping_detail_street'],
    'SHIPTOCITY': data['shipping_detail_city'],
    'SHIPTOSTATE': data['shipping_detail_state'],
    'SHIPTOZIP': data['shipping_detail_postcode'],
    'SHIPTOCOUNTRY': data['shipping_detail_country'],
  }
  trans['transactionData'] = {
    'CREDICCARDTYPE': data['card_type'].upper(),
    'ACCT': data['card_number'].replace(' ', ''),
    'EXPDATE': str(data['card_expiry_month'] + data ['card_expiry_year']),
    'CW2': data['card_ccv'],
    'AMT': trans['amount'],
    'INVNUM': str(order.id)
  }
  
  part1 = urlencode(trans['configuration']) + "&"
  part2 = "&" + urlencode(trans['custBillData'])
  part3 = "&" + urlencode(['custShipData'])
  trans['postString'] = (part1 + urlencode(trans['transactionData']) +
      part2 + part3)
  trans['postString'] = trans['postString'].encode('utf-8')
  request_args = {"url": trans['connection'], "data": trans['postString']}
  try:
    all_requests = urlopen(Request(**request_args)).read()
  except URLError:
    raise CheckoutError("Could not talk to PayPal payment getway")
  parsed_results = QueryDict(all_results)
  state = parsed_results['ACK']
  if state not in ["Success", "SuccessWithWarning"]:
    raise CheckoutError(parsed_results['L_LONGMESSAGE0'])
  return parsed_results['TRANSACTIONID']

COUNTRIES = (
  ("US", "UNITTED STATES"),
  ("CA", "CANADA"),
  (),
)
```

```
```

```
```
