When using curl, we can use the -i or --include options to include response headers in the output. If we only want the headers, we can use -I or --head.

Let's try using curl with -I to get the server headers on port 80 of the enumeration sandbox.

curl -I http://enum-sandbox

We can use this information to refine our enumeration and attack payloads. However, developers can modify or disable this header, so we can't always trust this value when we discover it without additional verification.
