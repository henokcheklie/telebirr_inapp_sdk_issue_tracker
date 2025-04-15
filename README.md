# ✨ Telebirr In-App SDK for Flutter

![Pub Version](https://img.shields.io/pub/v/telebirr_inapp_sdk)
![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)

Unofficial Flutter plugin that seamlessly integrates the Telebirr InApp payment SDK for Android and iOS platforms. This package eliminates the need for native implementation by providing a unified Flutter interface for Telebirr in-app payments.

## 🌟 Features

- Simplifies Telebirr in-app payment integration.
- Supports Android and iOS platforms.
- Provides an easy-to-use API for initiating and handling payments.

## 🎖 Installation

Add `telebirr_inapp_sdk` to your `pubspec.yaml` file:

```yaml
dependencies:
  telebirr_inapp_sdk: latest_version
```

Then, run:

```sh
flutter pub get
```

## ⚙️ Requirements

- Flutter: >=3.3.0
- Dart: >=3.0.0
- Android:
  - Minimum SDK (minSdk): 21 (Android 5.0 Lollipop)
  - Target/Compile SDK: 34 (Android 14)
  - Java Version: 17 (source and target compatibility)
  - Kotlin Version: 2.1.10
  - Gradle Plugin Version: 8.9.0
- iOS:
  - iOS 12.0 or higher

## 🖥️ Supported Platforms

| Platform | Supported |
| -------- | --------- |
| Android  | ✅        |
| iOS      | ✅        |
| Mac      | ❌        |
| Web      | ❌        |
| Linux    | ❌        |
| Windows  | ❌        |

## 🎮 Usage

```dart
import 'package:telebirr_inapp_sdk/telebirr_inapp_sdk.dart';

final telebirr = TelebirrInappSdk(
  appId: 'your_app_id',
  shortCode: 'your_short_code',
  receiveCode: 'your_receive_code',
);

// Start payment
try {
  final result = await telebirr.startPayment();
  if (result['success']) {
    print('Payment successful');
  } else {
    print('Payment failed: ${result['message']}');
  }
} catch (e) {
  print('Error: $e');
}
```

## 📥 Parameters

| Parameter   | Description                                                    |
| ----------- | -------------------------------------------------------------- |
| appId       | The App ID of the merchant, provided during payment setup.     |
| shortCode   | The Short Code of the merchant, provided during payment setup. |
| receiveCode | The code returned from your backend when placing a pre-order.  |

## 🔧 Platform-Specific Configuration

No additional configuration is required for both Android and iOS.

## 🖥️ Backend Integration

Implement the receive code fetch function:

```dart
Future<dynamic> _getReceiveCode({
  required String amount,
  required String title
}) async {
  var url = "YOUR_BACKEND_API_URL";
  Map data = {
    'amount': amount,
    'title': title,
  };

  try {
    http.Response response = await http.post(
      Uri.parse(url),
      headers: {
        "Content-Type": "application/json",
        "Accept": "application/json"
      },
      body: json.encode(data),
    ).timeout(
      Duration(seconds: 15),
      onTimeout: () {
        throw TimeoutException("The connection has timed out!");
      },
    );

    return json.decode(response.body);
  } catch (e) {
    print(e);
    return null;
  }
}
```

## 🎮 UI Implementation

Example UI for initiating a payment:

```dart
@override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(
            Platform.isIOS ? 'Telebirr SDK iOS Demo' : 'Telebirr SDK Demo'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(20.0),
        child: Form(
          key: _formKey,
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              Image.asset(
                "images/telebirr.png",
                height: 200,
                width: 200,
              ),
              const SizedBox(height: 30),
              TextFormField(
                controller: _amountController,
                keyboardType:
                    const TextInputType.numberWithOptions(decimal: true),
                decoration: InputDecoration(
                  labelText: 'Amount (ETB)',
                  hintText: 'Enter amount',
                  prefixIcon: const Icon(Icons.monetization_on),
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(8),
                  ),
                  errorMaxLines: 2,
                ),
                validator: _validateAmount,
                enabled: !_isProcessing,
                // Format input to allow only numbers and one decimal point
                inputFormatters: [
                  FilteringTextInputFormatter.allow(RegExp(r'^\d*\.?\d{0,2}')),
                ],
              ),
              const SizedBox(height: 8),
              if (_errorMessage.isNotEmpty)
                Container(
                  padding: const EdgeInsets.all(8),
                  decoration: BoxDecoration(
                    color: Colors.red.shade50,
                    borderRadius: BorderRadius.circular(8),
                  ),
                  child: Text(
                    _errorMessage,
                    style: TextStyle(color: Colors.red.shade700),
                  ),
                ),
              const SizedBox(height: 16),
              ElevatedButton(
                onPressed: _startPayment,
                style: ElevatedButton.styleFrom(
                  backgroundColor: Colors.black,
                  foregroundColor: Colors.white,
                  padding: const EdgeInsets.symmetric(vertical: 16),
                  shape: RoundedRectangleBorder(
                    borderRadius: BorderRadius.circular(8),
                  ),
                ),
                child: _isProcessing
                    ? const SpinKitThreeBounce(
                        color: Colors.white,
                        size: 24,
                      )
                    : const Text(
                        'Pay with Telebirr',
                        style: TextStyle(fontSize: 16),
                      ),
              ),
            ],
          ),
        ),
      ),
    );
  }
```

## 🧩 Complete Implementation Example

This is a complete example showing how to implement Telebirr payments with a user interface:

```dart
class PaymentScreen extends StatefulWidget {
  const PaymentScreen({super.key});

  @override
  State<PaymentScreen> createState() => _PaymentScreenState();
}

class _PaymentScreenState extends State<PaymentScreen> {
  final TextEditingController _amountController = TextEditingController();
  final _formKey = GlobalKey<FormState>();
  bool _isProcessing = false;
  String _errorMessage = '';
  String receiveCode = "";

  // Configuration constants
  static const String appId = "YOUR_APP_ID";
  static const String shortCode = "YOUR_SHORT_CODE";

  Future<void> _startPayment() async {
    if (!_formKey.currentState!.validate()) {
      return;
    }
    try {
      double amount = double.parse(_amountController.text);
      if (amount <= 0) {
        setState(() {
          _errorMessage = 'Amount must be greater than 0';
        });
        return;
      }

      setState(() {
        _isProcessing = true;
        _errorMessage = '';
      });

      // First get the receiveCode from your API
      final receiveCodeResult = await _getReceiveCode(
        amount: _amountController.text,
        title: "Telebirr Payment",
      );
      if (receiveCodeResult != null &&
          receiveCodeResult['createOrderResult']['result']
                  .toString()
                  .toLowerCase() ==
              'success') {
        setState(() {
          receiveCode = receiveCodeResult['createOrderResult']['biz_content']
              ['receiveCode'];
        });

        // Initialize the SDK and start payment
        final sdk = TelebirrInappSdk(
          appId: appId,
          shortCode: shortCode,
          receiveCode: receiveCode,
        );

        final result = await sdk.startPayment();

        // Always set processing to false when we get a response
        setState(() {
          _isProcessing = false;
        });

        if (!mounted) return;

        // Show appropriate message based on status
        bool isSuccess = (result.isNotEmpty && result["status"] == true);
        ScaffoldMessenger.of(context).showSnackBar(SnackBar(
          content: Text(
              isSuccess
                  ? "Payment completed successfully!"
                  : "Payment failed due to: ${result['message']}",
              style: TextStyle(color: Colors.white)),
          backgroundColor: isSuccess ? Colors.green : Colors.red,
        ));
      } else {
        setState(() {
          _isProcessing = false;
          _errorMessage =
              receiveCodeResult?['message'] ?? 'Failed to get receive code';
        });
      }
    } catch (e) {
      setState(() {
        _isProcessing = false;
        _errorMessage = "An error occurred: ${e.toString()}";
      });
    }
  }
}
```

## ❗ Error Codes

## 🔹 Telebirr SDK Error Codes

| Error Code | Description                                                                                     |
| ---------- | ----------------------------------------------------------------------------------------------- |
| 0          | Payment success.                                                                                |
| -1         | Unknown error.                                                                                  |
| -2         | Parameter error.                                                                                |
| -3         | Payment was cancelled by the user.                                                              |
| -10        | The Telebirr Payment app is not installed. (Telebirr super app is not installed on user device) |
| -11        | The installed version of Telebirr Payment does not support this function.                       |

## 🔸 Custom Plugin Error Codes

| Error Code | Description                                               |
| ---------- | --------------------------------------------------------- |
| -997       | Invalid response from payment plugin.                     |
| -996       | Generic error code (fallback if e.details is not an int). |
| -995       | Unexpected error, e.g., runtime exceptions.               |

## 📄 License

This project is licensed under the MIT License.

- see the [LICENSE](https://github.com/henokcheklie/telebirr_inapp_sdk_issue_tracker.git) file for details.
  > Note: This is a private project maintained by Henok Cheklie. Contributions are currently not accepted.

## 📬 Contact Information

- **Author**: Henok Cheklie
- **Email**: henokcheklie@gmail.com
- **Package**: telebirr_inapp_sdk
- **Repository**: Private GitHub Repository (Contributions are not accepted as this is a privately maintained project)

## 🐞 Issues and Feedback

Please file issues, bugs, or feature requests in our [issue tracker](https://github.com/henokcheklie/telebirr_inapp_sdk_issue_tracker).
