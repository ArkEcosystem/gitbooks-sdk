# Minimalistic SDK Demos

The code represents minimal example of `client` and `crypto` libraries usage for the specified programming language. Example functionality consists of:

* importing/loading the needed dependencies/libraries
* initialisation of the client and connecting to an ark-node \(peer\)
* retrieve a specific block via API
* create transaction payload
* post transaction payload to an ark-node \(peer\)
* handle response data from API

Please refer to the code comments or check more detailed documentation for specific SDK in the left menu.

### Java

```java
import com.google.gson.internal.LinkedTreeMap;
import org.arkecosystem.client.Connection;
import org.arkecosystem.client.api.two.Two;
import org.arkecosystem.crypto.configuration.Network;
import org.arkecosystem.crypto.networks.Devnet;
import org.arkecosystem.crypto.networks.Mainnet;
import org.arkecosystem.crypto.transactions.Transaction;
import org.arkecosystem.crypto.transactions.builder.Transfer;

import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;

public class Main {

    public static Transaction CreateTransferTransaction(int amount, String recipientAddress, String passphrase1) {
        return new Transfer()
                .recipient(recipientAddress)
                .amount(amount)
                .vendorField("This is a transaction from Java")
                .sign(passphrase1)
                .transaction;
    }

    public static void  ParseResponse(LinkedTreeMap<String, Object> postResponse) {
        // simple parsing from response object
        LinkedTreeMap<String, Object> responseData = (LinkedTreeMap<String, Object>) postResponse.get("data");
        LinkedTreeMap<String, Object> responseErrors = (LinkedTreeMap<String, Object>) postResponse.get("errors");

        ArrayList<String> accept = (ArrayList<String>) responseData.get("accept");
        ArrayList<String> broadcast = (ArrayList<String>) responseData.get("broadcast");
        ArrayList<String> invalid = (ArrayList<String>) responseData.get("invalid");
        ArrayList<String> excess = (ArrayList<String>) responseData.get("excess");

        System.out.println("Accepted: " + accept);
        System.out.println("Broadcasted: " + broadcast);
        System.out.println("Invalid: " + invalid);
        System.out.println("Excess: " + excess);
        System.out.println("Errors: " + responseErrors);
    }

    public static boolean CheckIfTrue(String transactionId, String key, LinkedTreeMap<String, Object> postResponse) {
        return  ((ArrayList<String>)((LinkedTreeMap<String, Object>) postResponse.get("data")).get(key)).contains(transactionId);
    }

    public static void main(String[] args) throws IOException {
        // setting the network, defaults to Mainnet
        // Network.set(new Mainnet());

        HashMap<String, Object> map = new HashMap<>();
        map.put("host", "http://node-ip:port/api/");
        map.put("API-Version", 2);

        Connection<Two> connection = new Connection(map);

        // testing blocks endpoint // find block with height 545774
        LinkedTreeMap<String, Object> actual = connection.api().blocks.show("545774");
        System.out.println(actual);

        // creating a transaction with the arkecosystem-java-crypto builder
        Transaction transfer1 = CreateTransferTransaction(1, "AJDkvwkwxW4N9bTYqJUijmEtY3rKwniCZQ", "secret passphrase");
        System.out.println(transfer1.toJson());

        // adding transaction to payload, payload is an array of transactions
        ArrayList<HashMap> payload = new ArrayList();
        payload.add(transfer1.toHashMap());

        // posting transactions to the connected node as specified in the connection above
        LinkedTreeMap<String, Object> postResponse = connection.api().transactions.create(payload);
        System.out.println("Raw response: " + postResponse);

        // calling ParseResponse
        ParseResponse(postResponse);

        // checking if transfer1 was accepted and broadcasted
        System.out.println("Accepted status: " + CheckIfTrue(transfer1.id,"accept", postResponse));
        System.out.println("Broadcast status: " + CheckIfTrue(transfer1.id,"broadcast", postResponse));
    }
}
```

### PHP

```php
<?php

require 'vendor/autoload.php';

use ARKEcosystem\Crypto\Transactions\Builder\Transfer;
use ARKEcosystem\Client\Connection;

// Instantiate a client
$connection = new Connection([
    'host' => 'http://node-ip:port/api/', // TRAILING SLASH!
]);

// find block with height 545774
$response = $connection->blocks()->show('545774');
var_dump($response['data']);

// creating a transaction with the arkecosystem-java-crypto builder
$transfer = Transfer::new()
    ->recipient('AJDkvwkwxW4N9bTYqJUijmEtY3rKwniCZQ')
    ->amount(1)
    ->vendorField('This is a transaction from PHP')
    ->sign('secret passphrase');

// adding transaction to payload, payload is an array of transactions
$transactions = [$transfer->toArray()];

// posting transactions to the connected node as specified in the connection above
$response = $connection->transactions()->create($transactions);

var_dump($response['data']);
var_dump($response['data']['accept']);
var_dump($response['data']['broadcast']);
```

### Elixir

```elixir
defmodule ElixirTest do

  alias ARKEcosystem.Crypto.Transactions.Transaction
  alias ARKEcosystem.Crypto.Transactions.Builder

  @client ARKEcosystem.Client.new(%{
          host: "https://dexplorer.ark.io:8443/api/v2",
          nethash: "6e84d08bd299ed97c212c886c98a57e36545c8f5d645ca7eeae63a8bd62d8988",
          version: "2.0.0"
  })

  def main do
    #Find a block with height 939627
    IO.inspect ARKEcosystem.Client.API.Two.Block.show(@client, '939627')

    transaction = Builder.build_transfer(
      "D5rHMAmTXVbG7HVF3NvTN3ghpWGEii5mH2",
      100000,
      "This is a transaction from Elixir",
      "my super secret seedpass"
    )

    # adding transaction to payload, payload is an array of transactions
    transaction = transaction |> Transaction.to_params

    # posting transactions to the connected node as specified in the connection above
    ARKEcosystem.Client.API.Two.Transaction.create(@client, [transaction])
  end
end
```

### Python

```python
from ark.client import ARKClient
from crypto.transactions.builder.transfer import Transfer

# Instantiate the Client
client = ARKClient('https://dexplorer.ark.io:8443/api/', api_version='v2')

# Find Block With Height 545774
response = client.blocks.get('545774')

# Creating a Transaction With the ARKecosystem-Python-Crypto Builder
tx = Transfer(recipientId='D5rHMAmTXVbG7HVF3NvTN3ghpWGEii5mH2', amount=10000000,
              vendorField="This is a transaction from Python")

tx.sign('my super secret seedpass')

# adding transaction to payload, payload is an array of transactions
transaction_dict = tx.to_dict()


if __name__ == '__main__':
    # posting transactions to the connected node as specifid in the connection above
    client.transactions.create([transaction_dict])
```

### C++

#### **Arduino \(Adafruit ESP32 Feather\)**

```cpp
#include <Arduino.h>

//  include the 'arkClient.h' and 'arkCrypto.h' headers.
#include <arkClient.h>
#include <arkCrypto.h>

#include <stdio.h>  //  included for the 'snprintf' method.
#include <string>

//  'WiFi.h' and 'HTTPClient.h' are ESP32-specific headers.
#include <WiFi.h>
#include <HTTPClient.h>

#include "time.h"   //  for configuring the ESP32's time.

const char* ssid = "yourWiFiSSID";
const char* password = "yourWiFiPassword";


void doExample() {
    //  create the connection.
    Ark::Client::Connection<Ark::Client::Api> connection("167.114.29.55", 4003);

    //  retrieve a specific block.
    std::string blockResponse = connection.api.blocks.get("58328125061111756");

    //  create transaction payload.
    auto transfer = Ark::Crypto::Transactions::Builder::buildTransfer(
        "recipientID",
        1000000000,
        "vendorfield",
        "passphrase",
        "secondPassphrase");

    //  post transaction payload to an ark-node (peer).
    char transactionsBuffer[600];
    snprintf(&transactionsBuffer[0], 600, "{\"transactions\":[%s]}", transfer.toJson().c_str());

    // Let's save the transactionsBuffer to a 'string' object for passing to Cpp-Client.
    std::string jsonStr = transactionsBuffer;

    std::string sendResponse = connection.api.transactions.send(jsonStr);

    //  handle response data from API.
    Serial.println(sendResponse.c_str());
};

void setup() {
    //  start your serial connection.
    Serial.begin(115200);

    //  connect your board to WiFi.
    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }

    //  Configure your boards time server.
    configTime(0, 0, "pool.ntp.org");

    doExample();
};

void loop() { };
```

#### **PlatformIO \(Adafruit ESP32 Feather\)**

**platformio.ini**

```text
; PlatformIO Project Configuration File
;
;   Build options: build flags, source filter
;   Upload options: custom upload port, speed and extra flags
;   Library options: dependencies, extra library storages
;   Advanced options: extra scripting
;
; Please visit documentation for the other options and examples
; https://docs.platformio.org/page/projectconf.html

[env:featheresp32]
platform = espressif32
board = featheresp32
framework = arduino
lib_deps = ARK-Cpp-Client, ARK-Cpp-Crypto
upload_speed = 921600
monitor_speed = 115200
```

**main.cpp**

```cpp
#include <Arduino.h>

//  include the 'arkClient.h' and 'arkCrypto.h' headers.
#include <arkClient.h>
#include <arkCrypto.h>

#include <stdio.h>  //  included for the 'snprintf' method.
#include <string>

//  'WiFi.h' and 'HTTPClient.h' are ESP32-specific headers.
#include <WiFi.h>
#include <HTTPClient.h>

#include "time.h"   //  for configuring the ESP32's time.

const char* ssid = "yourWiFiSSID";
const char* password = "yourWiFiPassword";


void doExample() {
    //  create the connection.
    Ark::Client::Connection<Ark::Client::Api> connection("167.114.29.55", 4003);

    //  retrieve a specific block.
    std::string blockResponse = connection.api.blocks.get("58328125061111756");

    //  create transaction payload.
    auto transfer = Ark::Crypto::Transactions::Builder::buildTransfer(
        "recipientID",
        1000000000,
        "vendorfield",
        "passphrase",
        "secondPassphrase");

    //  post transaction payload to an ark-node (peer).
    char transactionsBuffer[600];
    snprintf(&transactionsBuffer[0], 600, "{\"transactions\":[%s]}", transfer.toJson().c_str());

    // Let's save the transactionsBuffer to a 'string' object for passing to Cpp-Client.
    std::string jsonStr = transactionsBuffer;

    std::string sendResponse = connection.api.transactions.send(jsonStr);

    //  handle response data from API.
    Serial.println(sendResponse.c_str());
};

void setup() {
    //  start your serial connection.
    Serial.begin(115200);

    //  connect your board to WiFi.
    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }

    //  Configure your boards time server.
    configTime(0, 0, "pool.ntp.org");

    doExample();
};

void loop() { };
```

**Desktop/Server**

```cpp
//  include the 'arkClient.h' and 'arkCrypto.h' headers.
#include <arkClient.h>
#include <arkCrypto.h>

#include <iostream> //  used to print transaction response to the console.
#include <stdio.h>  //  included for the 'snprintf' method.
#include <string>


int main() {
    //  create the connection.
    Ark::Client::Connection<Ark::Client::Api> connection("167.114.29.55", 4003);

    //  retrieve a specific block.
    std::string blockResponse = connection.api.blocks.get("58328125061111756");

    //  create transaction payload.
    auto transfer = Ark::Crypto::Transactions::Builder::buildTransfer(
        "recipientID",
        1000000000,
        "vendorfield",
        "passphrase",
        "secondPassphrase");

    //  post transaction payload to an ark-node (peer).
    char transactionsBuffer[600];
    snprintf(&transactionsBuffer[0], 600, "{\"transactions\":[%s]}", transfer.toJson().c_str());

    std::string sendResponse = connection.api.transactions.send(transactionsBuffer);

    //  handle response data from API.
    std::cout << sendResponse;
}
```

### Ruby

```ruby
require 'arkecosystem/client'
require 'arkecosystem/crypto'

connection = ARKEcosystem::Client::Connection.new(host: "http://my.ark.node:4003/api/", version: 2)

transaction = ARKEcosystem::Crypto::Transactions::Builder::Transfer.new()
                  .set_recipient_id('DBk4cPYpqp7EBcvkstVDpyX7RQJNHxpMg8')
                  .set_amount(100000)
                  .set_vendor_field('Hello from Ruby !')
                  .sign('mysupersecretppassphrase')

puts transaction.verify

response = connection.transactions.create({transactions: [transaction.to_params]})

puts response.body
```

### Swift

```swift
import UIKit
import SwiftClient
import SwiftCrypto

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        // Initiate the client
        let conn = Connection(host: "https://dexplorer.ark.io:8443/api")

        // Set the correct network, devnet is default but explicitly set here
        ARKNetwork.shared.set(network: Devnet())

        // Create a transfer transaction
        let transfer = ARKBuilder.buildTransfer("secret passphrase",
                                                secondPassphrase: nil,
                                                to: "DBk4cPYpqp7EBcvkstVDpyX7RQJNHxpMg8",
                                                amount: 10000000,
                                                vendorField: "this is a tx from Swift")

        // Send the transaction
        let transactions = Transactions(connection: conn)
        transactions.create(body: ["transactions": [transfer.toDict()]]) { (response) in
            print(response)
        }
    }
}
```

### Go

```go
package main

import (
    "context"
    arkClient "github.com/arkecosystem/go-client/client"
    arkCrypto "github.com/arkecosystem/go-crypto/crypto"
    "github.com/davecgh/go-spew/spew"
    "net/url"
)

func main() {
    // Initiate the client
    connection := arkClient.NewClient(nil)
    url, _ := url.Parse("http://my.ark.node.ip:port/api/")
    connection.BaseURL = url


    // Find a block at height 939627
    responseStructBlock, _, _ := connection.Blocks.Get(context.Background(), 939627)

    spew.Dump(responseStructBlock)

    // Create a transfer transaction
    transaction := arkCrypto.BuildTransfer("arkAddress", 100000, "Hello World", "mysupersecretppassphrase", "mysupersecretsecondppassphrase")

    body := &arkClient.CreateTransactionRequest{
        Transactions: []interface{}{
            *transaction,
        },
    }

    // Send the transaction
    connection.Transactions.Create(context.Background(), body)
}
```

