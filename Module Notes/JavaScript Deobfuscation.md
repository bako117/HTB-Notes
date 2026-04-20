The ability to deobfuscate code is a useful technique that can be applied to various real-world scenarios. It is useful on web application assessments to determine if a developer has used "security by obscurity" to hide JavaScript code containing sensitive data. It can also be useful for defenders when, for example, attempting to deobfuscate code that was responsible for the Phishing website used in an attack.

The following topics will be covered in these notes:
- Locating JavaScript code
- Intro to Code Obfuscation
- How to Deobfuscate JavaScript code
- How to decode encoded messages
- Basic Code Analysis
- Sending basic HTTP requests
# Intro
## JavaScript

Most sites use JavaScript to perform their functions. 
- HTML is used for the main fields and parameters
- CSS is used to determine its design and format
- JavaScript (JS) is used to perform necessary functions to run the site, often in the background

JS can be written internally between `<script>` tags or a sperate .js file can be referenced. 

```html
<script src="secret.js"></script>
```

In the case above, we can click the script and it should take us to the code. Sometime code can be very obfuscated. 

## Obfuscation

Obfuscation is a technique used to make a script more difficult to read by humans but allows it to function the same from a technical point of view, though performance may be slower.

This is normally done with an obfuscation tool. 

Codes written in many languages are published and executed without being compiled in `interpreted` languages, such as `Python`, `PHP`, and `JavaScript`. JS normally resides client-side, making it more vulnerable. 

Developers might legitimately want to obfuscate because:
- They don't want others taking their code
- They want an additional layer of security

An attacker would obfuscate to: 
- Hide IOC's 
- Hide malicious intent
- Avoid detection

#  JS Obfuscation Levels
## Basic 

A common way of reducing the readability of a snippet of JavaScript code while keeping it fully functional is JavaScript minification. `Code minification` means having the entire code in a single (often very long) line.

https://beautifytools.com/javascript-obfuscator.php is another quick way to obfuscate JS. 

A `packer` obfuscation tool usually attempts to convert all words and symbols of the code into a list or a dictionary and then refer to them using the `(p,a,c,k,e,d)` function to re-build the original code during execution.

## Difficult

 [https://obfuscator.io](https://obfuscator.io/). Before we click `obfuscate`, we will change `String Array Encoding` to `Base64`. The more obfuscated the code gets, the longer it can take to run. Super obfuscated code should be reserved for bypassing web filters or restrictions. 

NOTE: there are other tools for obfuscating JavaScript btw. 

# Deobfuscation 
Now that we understand obfuscation, we can work towards reversing it. There are tools to help us deobfuscate this as well.

For example, if we were using Firefox, we can open the browser debugger with [ `CTRL+SHIFT+Z` ], and then click on our script `secret.js`. This will show the script in its original formatting, but we can click on the '`{ }`' button at the bottom, which will `Pretty Print` the script into its proper JavaScript formatting. 

Furthermore, we can utilize many online tools or code editor plugins, like [Prettier](https://prettier.io/playground/) or [Beautifier](https://beautifier.io/). Let us copy the `secret.js` script:

We can find many good online tools to deobfuscate JavaScript code and turn it into something we can understand. One good tool is [UnPacker](https://matthewfl.com/unPacker.html). Let's try copying our above-obfuscated code and run it in UnPacker by clicking the `UnPack` button.

## Code Analysis

Once we deobfuscate code, we can start doing static analysis. 

If we want to encode anything or decode anything in Linux with base64 we will do | base64 or | base64 -d. 