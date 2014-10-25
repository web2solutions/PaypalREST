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
	
	my $payment = $paypal->pay_with_ballance({
	        amount	=> $amount,
	        description => $pay_for_desc,
	        return_url => $PaymentFlowPath . '/processors/paypal_return.pl?invoice_id=' . $invoice_id,
	        cancel_url => $PaymentFlowPath . '/processors/paypal_cancel.pl?invoice_id=' . $invoice_id
	}); 
````
