PaypalREST
===========

PaypalREST - Consume Paypal REST API via perl

## INSTALL

$ cpan App::cpanminus

$ cpanm git://github.com/web2solutions/PaypalREST.git

## SYNOPSIS

````perl
	use PaypalREST;
	
	my $paypal = PaypalREST->new(
		$client_id
		,$client_secret
	);
````

**Pay with Paypal balance**
````perl	
	my $payment = $paypal->pay_with_ballance({
	        amount	=> 1.59,
	        description => 'pay for service',
	        return_url => 'your_return_url',
	        cancel_url => 'your_cancel_url'
	}); 
````

**Pay with Credit card balance**

````perl	
	my $payment = $paypal->pay_as_guest({
	        number       => '4353185781082049',
	        type         => 'visa',
	        cvv2           => '442',
	        expire_month => 3,
	        expire_year  => 2018,
	        amount          => 3.58,
	        first_name  => 'Steve',
	        last_name  => 'Jobs',
	        line1  => 'Street 345',
	        city  => 'Palo Alto',
	        state  => 'CA',
	        postal_code  => '4444',
	        country_code  => 'US',
	        description => 'pay for service'
	});
````
