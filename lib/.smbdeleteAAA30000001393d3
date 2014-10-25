package PaypalREST;

use strict;
use JSON;
use LWP;
use Crypt::CBC;
use Carp ("croak");
use Cache::FileCache;

our $VERSION = '0.001';

our $ENDPOINT_SANDBOX = "https://api.sandbox.paypal.com";
our $ENDPOINT_LIVE    = "https://api.paypal.com";

my $live = 0;

my $json = JSON->new->allow_nonref;

my $last_error;

sub live {
    my $class = shift;
    my ($value) = @_;
    return $live = $value;
}

sub endpoint {
    my $class = shift;

    if ( $live == 1 ) {
        return $ENDPOINT_LIVE;
    }
    return $ENDPOINT_SANDBOX;
}

sub new {
    my $class = shift;

    my %args = (
        client_id    => $_[0],
        secret       => $_[1],
        user_agent   => LWP::UserAgent->new,
        app_id       => undef,
        access_token => undef,
        @_
    );

    unless ( $args{client_id} && $args{secret} ) {
        croak " new() : client_id and secret are missing ";
    }

    #
    # checking if access_token is available from previous requests
    #
    my $cache = Cache::FileCache->new( { cache_root => File::Spec->tmpdir, namespace => 'PaypalREST' } );

    my $cipher = Crypt::CBC->new( -key => $args{"secret"}, -cipher => 'Blowfish' );

    if ( my $e_token = $cache->get( $args{"client_id"} ) ) {
        $args{access_token} = $cipher->decrypt($e_token);
    }

    else {

        # if access_token cannot be found in the cache we need to authenticate ourselves to get one
        my $ua = $args{user_agent};

        my $h = HTTP::Headers->new(
            Accept            => "application/json",
            'Accept-Language' => 'en_US',
            'content-type' => 'application/x-www-form-urlencoded'
        );

        $h->authorization_basic( $args{client_id}, $args{secret} );

        my $endpoint = $class->endpoint;

        my $req = HTTP::Request->new( "POST", $endpoint . '/v1/oauth2/token', $h );
        $req->content("grant_type=client_credentials");

        my $res = $ua->request($req);
        unless ( $res->is_success ) {
            croak "Authorization failed : " . $res->status_line . ', ' . $res->content;
        }

        my $res_hash = _json_decode( $res->content );

        $args{access_token} = $res_hash->{access_token};
        $args{app_id}       = $res_hash->{app_id};

        $cache->set( $args{"client_id"}, $cipher->encrypt( $args{access_token} ), $res_hash->{expires_in} - 5 );
    }
    return bless( \%args, $class );
}



sub _json_decode {
    my $text = shift;
    my $hashref;
    eval { $hashref = $json->decode($text); };

    if ( my $error = $@ ) {
        croak "_json_decode(): cannot decode $text: $error";
    }
    return $hashref;
}

sub _json_encode {
    my $hashref = shift;
    return $json->encode($hashref);
}

sub rest {
    my $self = shift;
    my ( $method, $path, $json, $dump_responce ) = @_;

    unless ( $path =~ /\/$/ ) {
        $path = $path . '/';
    }

    my $endpoint = $self->endpoint;
    $endpoint = sprintf( " % s%s", $endpoint, $path );
    my $a_token = $self->{access_token};
    my $req = HTTP::Request->new( $method, $endpoint, [ 'Content-Type', 'application/json', 'Authorization', "Bearer $a_token" ] );

    if ($json) {
        $req->content($json);
    }

    my $ua  = $self->{user_agent};
    my $res = $ua->request($req);

    if ($dump_responce) {
        require Data::Dumper;
        return Data::Dumper::Dumper($res);
    }

    unless ( $res->is_success ) {
        if ( my $content = $res->content ) {
            my $error = _json_decode( $res->content );
            $self->error( sprintf( "%s: %s. See: %s", $error->{name}, $error->{message}, $error->{information_link} ) );
            return undef;
        }
        $self->error( $res->status_line );
        return undef;
    }
    return _json_decode( $res->content );
}


sub pay_with_ballance {
    my $self = shift;
    my ($data) = @_;

    foreach my $field (qw/amount return_url cancel_url description/) {
        unless ( $data->{$field} ) {
            croak "payment(): $field is a required field";
        }
    }


    my $request_hash = {
        intent => 'sale',
        redirect_urls => {
            return_url => $data->{return_url},
            cancel_url => $data->{cancel_url}
        },    
        payer  => {
            payment_method      => "paypal"
        },
        transactions => [
            {
                amount => {
                    total    => $data->{amount},
                    currency => $data->{currency} || "USD"
                },
                description    => $data->{description}
            }
        ]
    };

    if ( $data->{redirect_urls} ) {
        $request_hash->{redirect_urls} = $data->{redirect_urls};
    }

    return $self->rest( 'POST', "/v1/payments/payment", _json_encode($request_hash) );
}


sub execute_payment {
    my $self = shift;
    my ($data) = @_;

    foreach my $field (qw/execute_url payer_id/) {
        unless ( $data->{$field} ) {
            croak "payment(): $field is a required field";
        }
    }

    my $request_hash = {
        payer_id => $data->{payer_id}
    };

    if ( $data->{redirect_urls} ) {
        $request_hash->{redirect_urls} = $data->{redirect_urls};
    }
    
    return $self->rest( 'POST', $data->{execute_url}, _json_encode($request_hash) );
}

sub pay_as_guest {
    my $self = shift;
    my ($data) = @_;

    foreach my $field (qw/number amount type cvv2 description expire_month expire_year first_name last_name line1 city state postal_code/) {
        unless ( $data->{$field} ) {
            croak "payment(): $field is a required field";
        }
    }

    my %credit_card = (
        
        #number       => '4353185781082049',
        #type         => 'visa',
        #expire_month => 3,
        #expire_year  => 2018
        #first_name      => 'Sherzod',
        #last_name       => 'Ruzmetov',
        #amount          => 19.95,        

        number       => $data->{number},
        type         => $data->{type},
        cvv2           => $data->{cvv2},
        expire_month => $data->{expire_month},
        expire_year  => $data->{expire_year},
        first_name  => $data->{first_name},
        last_name  => $data->{last_name},
        billing_address => {
            line1  => $data->{line1},
            city  => $data->{city},
            state  => $data->{state},
            postal_code  => $data->{postal_code},
            country_code  => $data->{country_code} || 'US'
        }
    );

    my $request_hash = {
        intent => 'sale',
        payer  => {
            payment_method      => "credit_card",
            funding_instruments => [ { credit_card => \%credit_card } ]
        },
        transactions => [
            {
                amount => {
                    total    => $data->{amount},
                    currency => $data->{currency} || "USD"
                },
                description    => $data->{description}
            }
        ]
    };

    #return $request_hash;
    #exit;

    if ( $data->{redirect_urls} ) {
        $request_hash->{redirect_urls} = $data->{redirect_urls};
    }

    return $self->rest( 'POST', "/v1/payments/payment", _json_encode($request_hash) );
}

sub stored_cc_payment {
    my $self = shift;
    my ($data) = @_;

    unless ( $data->{id} ) {
        croak "stored_cc_payment(): 'id' is missing";
    }

    my $request_hash = {
        intent => 'sale',
        payer  => {
            payment_method      => "credit_card",
            funding_instruments => [ { credit_card_token => { credit_card_id => $data->{id} } } ]
        },
        transactions => [
            {
                amount => {
                    total    => $data->{amount},
                    currency => $data->{currency} || "USD"
                },
            }
        ]
    };

    if ( $data->{redirect_urls} ) {
        $request_hash->{redirect_urls} = $data->{redirect_urls};
    }

    return $self->rest( 'POST', "/v1/payments/payment", _json_encode($request_hash) );
}

sub get_payment {
    my $self = shift;
    my ($id) = @_;

    unless ($id) {
        croak "get_payment(): Invalid Payment ID";
    }

    return $self->rest( "GET", "/v1/payments/payment/$id" );
}

sub get_payments {
    my $self = shift;

    return $self->rest( "GET", "/v1/payments/payment" );
}

sub store_cc {
    my $self = shift;
    my ($data) = @_;

    my %credit_card = (
        number       => $data->{number}       || $data->{cc_number},
        type         => $data->{type}         || $data->{cc_type},
        expire_month => $data->{expire_month} || $data->{cc_expire_month},
        expire_year  => $data->{expire_year}  || $data->{cc_expire_year}
    );

    if ( my $cvv2 = $data->{cvv2} || $data->{cc_cvv2} ) {
        $credit_card{cvv2} = $cvv2;
    }

    foreach my $field (qw/first_name last_name billing_address/) {
        if ( $data->{$field} ) {
            $credit_card{$field} = $data->{$field};
        }
    }
    return $self->rest( 'POST', "/v1/vault/credit-card", _json_encode( \%credit_card ) );
}

sub get_cc {
    my $self = shift;
    my ($id) = @_;
    return $self->rest( "GET", "/v1/vault/credit-card/$id" );
}

sub error {
    my $self = shift;
    my ($new_message) = @_;

    unless ($new_message) {
        return $last_error;
    }

    $last_error = $new_message;
}

1;
__END__

=head1 NAME

PaypalREST - Consume Paypal REST API via perl

=head1 SYNOPSIS

    use PaypalREST;
    my $paypal = PaypalREST->new(
		$client_id
		,$client_secret
	);
