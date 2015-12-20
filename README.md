QBit::Application::Model::Authorization
=====

Simple model for authorization in QBit application.

## Usage

Install:

```
dpk-apt get install libqbit-application-model-authorization
```

Require:

```
use QBit::Application::Model::Authorization accessor => 'session'; # in Application.pm
```

### Methods:

  - registration - add new row into authorization table and return session id

```
my $session = $app->session->registration(['name-surname@mail.ru', 'login'], 'password');
```

  - check_auth - return session id

```
my $session = $app->session->check_auth('name-surname@mail.ru', 'password');
my $session = $app->session->check_auth('login', 'password');
```

  - check_session - check session id and return key

```
my $key = $app->session->check_session($session);
```

  - delete - delete row from authorization table

```
$app->session->delete($key);
```

  - process - combine all methods. (Expected cookie name where saved session, interval save session, Request, Response and call back function)
  
```
sub pre_cmd {
    my ($self, $controller) = @_;
    
    ...
    
    my $auth;
    try {
        $auth = $self->session->process('session_name', '+2d', $self->request, $self->response, sub {
            my $cur_user = $self->users->get_all(
                fields => [qw(id login name mail)],
                filter => ['OR', [{login => $_[0]}, {mail => $_[0]}]]
            )->[0];
            
           $self->set_option('cur_user' => $cur_user);
        });
    } catch {
        $auth = FALSE;
    };
    
    unless ($auth) {
        $controller->redirect('authorization', path => 'users');
        return $self->break();
    }
    
    ...
    
}
```