# Beam URI scheme
## EBNF syntax
` beam_urn     = "beam:" hex_address ["?" params]`

 `hex_address  = base16+`

 `params       = param ["&" params]`

 `param        = amount_param`

 `amount_param = "amount=" digit+ [ "." digit* ]`

## Simplified syntax

`beam: <address> [?amount=<amount>]`

