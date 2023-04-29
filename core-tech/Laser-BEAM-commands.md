1) show laser channels

`./beam-wallet-masternet laser --laser_channels_list`

2) wait incoming laser connection (will generate address)

`./beam-wallet-masternet laser --laser_receive --laser_my_locked_amount <amount in beam> --laser_remote_locked_amount <amount in beam> --laser_fee <amount in groth>`

Example:

`./beam-wallet-masternet laser --laser_receive --laser_my_locked_amount 1.1 --laser_remote_locked_amount 1.1 --laser_fee 100`

3) connect with laser

`./beam-wallet-masternet laser --laser_open --laser_address <address> --laser_my_locked_amount <amount in beam> --laser_remote_locked_amount <amount in beam> --laser_fee <amount in groth>`

Example:

`./beam-wallet-masternet laser --laser_open --laser_address 285a776d78e6e0ee285a196282e61768b87c7c108a7d8cf7622a094555d2cfeb80e --laser_my_locked_amount 0.9 --laser_remote_locked_amount 1.1 --laser_fee 100`

4) listen laser channels

`./beam-wallet-masternet laser --laser_listen [channel id 1,channel id 2, ... channel id N]`

Example:

`./beam-wallet-masternet laser --laser_listen`

`./beam-wallet-masternet laser --laser_listen 4bd5ee31b264f6102709dc145cf37b55`

`./beam-wallet-masternet laser --laser_listen 4bd5ee31b264f6102709dc145cf37b55,73e5af986eb3ea165f71bbb54ebfad37`

5) send coins

`./beam-wallet-masternet laser --laser_send <amount in beam> --laser_channel <channel id>`

Example:

`./beam-wallet-masternet laser --laser_send 0.1 --laser_channel 4bd5ee31b264f6102709dc145cf37b55`

6) close laser channels. after lock time is up or if other side is offline

`./beam-wallet-masternet laser --laser_drop <channel id 1,channel id 2, ... channel id N>`

Example:

`./beam-wallet-masternet laser --laser_drop 4bd5ee31b264f6102709dc145cf37b55`

`./beam-wallet-masternet laser --laser_drop 4bd5ee31b264f6102709dc145cf37b55,73e5af986eb3ea165f71bbb54ebfad37`

7) close laser channels. before lock time is up, only if other side is online

`./beam-wallet-masternet laser --laser_close <channel id 1,channel id 2, ... channel id N>`

Example:

`./beam-wallet-masternet laser --laser_close 4bd5ee31b264f6102709dc145cf37b55`

`./beam-wallet-masternet laser --laser_close 4bd5ee31b264f6102709dc145cf37b55,73e5af986eb3ea165f71bbb54ebfad37`

8) delete laser channels from DB, only for closed channels

`./beam-wallet-masternet laser --laser_delete <channel id 1,channel id 2, ... channel id N>`

Example:

`./beam-wallet-masternet laser --laser_delete 4bd5ee31b264f6102709dc145cf37b55`

`./beam-wallet-masternet laser --laser_delete 4bd5ee31b264f6102709dc145cf37b55,73e5af986eb3ea165f71bbb54ebfad37`