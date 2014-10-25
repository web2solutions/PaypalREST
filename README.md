PaypalREST
===========

PaypalREST - Consume Paypal REST API via perl

## INSTALL

$ cpan App::cpanminus

$ cpanm git://github.com/web2solutions/PaypalREST.git

## SYNOPSIS

        use PaypalREST;
    	my $paypal = PaypalREST->new(
		$client_id
		,$client_secret
	);

