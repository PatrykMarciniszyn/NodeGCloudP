
# Projekt Przetwarzanie w chmurach obliczeniowych

Jako zaliczenie projektu na wyżej wymieniony przedmiot wykonałem aplikację prostego sklepu internetowego.
Aplikacja ta w założeniach miała pokazać umiejętności korzystania z CRUD, zachowania bezpieczeństwa danych przy przesyłaniu zapytań do serwera / bazy dancyh oraz umiejętność wdrorzenia tego rozwiązania z wykorzystaniem chmur obliczeniowych.

# Dokumentacja Techniczna


Projekt składa się z trzech rozłącznych modułów.,

- [Klaster Bazy Danych](#Klaster-Bazy-Danych)
- [Backend Sever](#Backend-Server)
- [Frontend API](#Frontend-API)


## Klaster Bazy Danych

Baza danych oparta jest na technologi NoSQL z wykorzystaniem oprogramowania MongoDB. Baza danych jest tworzona dynamicznie po stronie servera. 
Projekt wykorzystuje prostą strukturę bazy danych. Składa się ona z jednej bazy danych o nazwie "Shop" w której umieszczone są dwie kolekcje:
* products
* users

W kolekcji products są przechowywane obiekty o strukturze: <br>

```json
_id : 5fb3bc25bda816248cea20fc
name : "Domek"
description : "Domek"
price :99999.99
image : "https://static.meblobranie.pl/userFiles/shop/items/img/25075/AGAWA_WIZ..."
```

Obiekty te służą do przechowoywania dancyh o przedmiotach wystawionych na sprzedarz.<br>

Druga kolekcja zawiera dane logowania użytkowników: <br>
```json 
_id : 5fb3b8aef116382d3c58601e
email : "a@a.pl"
password : "$2a$12$Fs3kc9wJ7r7pn5fGsmYzX.jVoeQ0ProDqxHSxWdgRo1AP8NUZIVhq"
```
Jako login został wykorzystany adres e-mail. Hasło dla bezpieczeństwa przed zapisaniem w bazie zostało zhashowane.

## Backend Server

Strona serverowa aplikacji została wykonana za pomocą technologi node.js.<br>
 Aplikacja ta została rozbina na strukturę plików:
 
 ```
my-app/
  README.md
  node_modules/
  package.json
  backend/
        routes/
            auth.js
            products.js
        app.js
        db.js
```

Moduł ten odpowiada za odbieranie zapytań interfejsu użytkownika, interpretiwaniu ich oraz wysyłaniu odpowiedzi. 

Na przykład: 

``` js 

router.get('/', (req, res, next) => {
  const queryPage = req.query.page;
  const pageSize = 1;
  const products = [];
  db.getDb()
    .db()
    .collection('products')
    .find()
    .sort({price: -1})
    .forEach(productDoc => {
      productDoc.price = productDoc.price.toString();
      products.push(productDoc);
    })
    .then(result => {
      res.status(200).json(products);
    })
    .catch(err => {
      console.log(err);
      res.status(500).json({ message: 'An error occurred.' });
    });
});

``` 

Ta przykładowa funkcja odpowiada na bezparametrowe zapytanie "*.com/products/". <br>
W odpowiedzi dostaje obiejk .json w postaci np: <br>

``` json
[
    {
        "_id":"5fc400c398d623000a2448f7",
        "name":"Obraz",
        "description":"Twój",
        "price":"9999999",
        "image":"https://t1.gstatic.com/images?q=tbn:ANd9GcTsyVprhLsmgKRGx3yhmZVjYbOFJLC2bG3G_urox5XXQ49qZDUMFDBMe9AqtC_V"
    },
    {
        "_id":"5fc4002a98d623000a2448f6",
        "name":"Witkacy  - Walka ",
        "description":"Obraz Witkacego\n",
        "price":"999999",
        "image":"https://upload.wikimedia.org/wikipedia/commons/4/4a/Stanis%C5%82aw_Ignacy_Witkiewicz_-_Walka_-_Google_Art_Project.jpg"
    },
    {
        "_id":"5fb3bc25bda816248cea20fc",
        "name":"Domek","description":"Domek dla ludzików \n",
        "price":"99999.99","image":"https://static.meblobranie.pl/userFiles/shop/items/img/25075/AGAWA_WIZ.4_min_1-110561-m.jpeg"
    },
    {
        "_id":"5fb3bf22bda816248cea20fe",
        "name":"Deska do prasowania ",
        "description":"Deska do prasowania niebieska\n",
        "price":"200.99",
        "image":"https://image.ceneostatic.pl/data/products/15308596/i-deska-do-prasowania-top-house-festival-125x48cm.jpg"
    }
] 
```
<br>
Odpowiedź ta jest przekazywana do Modułu Frontend API, interpretowana i wyświetlana dla użytkownika.
<br>

## Frontend API

Moduł interfejsu użytkownika został zaprogramowany z wykorzystaniem biblioteki javascript "React".
<br>

Jest to przykładowe wykonanie interfejsu użytkownika. Backend oraz frontend to dwa osobne mikroservisy niezależne od siebie. <br>

Serwis umożliwia w prosty sposób rozszerzanie możliwości całej aplikacji. Dodatkowe mikroserwist mogą korzystać z zapytań, na które odpowiada już serwer, a także można w prosty sposób rozszeżyć możliwości serwera oraz oprzeć na nim całkowice inny moduł kliencki współdziecąc jedynie np. moduł logowania. 

``` js
  fetchData = () => {
    axios
      .get('https://nodejsf.oa.r.appspot.com/products')
      .then(productsResponse => {
        this.setState({ isLoading: false, products: productsResponse.data });
      })
      .catch(err => {
        this.setState({ isLoading: false, products: [] });
        this.props.onError('Loading products failed. Please try again later');
        console.log(err);
      });
  };

  render() {
    let content = <p>Loading products...</p>;

    if (!this.state.isLoading && this.state.products.length > 0) {
      content = (
        <Products
          products={this.state.products}
          onDeleteProduct={this.productDeleteHandler}
        />
      );
    }
    if (!this.state.isLoading && this.state.products.length === 0) {
      content = <p>Found no products. Try again later.</p>;
    }
    return <main>{content}</main>;
  }
  ```

  Wyżej wymieniona funkcja jest przykładem pobierania i wykorzystywania danych pozyskanych z serwera do wyświetlenia użytkownikowi listy dostępnych przedmiotów. 


  # Dokumentacja Użytkownika

  Użytkownik aplikacji korzystając z interfejsu może wykonać szereg akcji:

  - [Rejstracja Użytkownia](#Rejstracja-Użytkownia)
  - [Logowanie Użytkownia](#Logowanie-Użytkownia)
  - [Dodawanie Produktu/Edycja](#Dodawanie-Produktu/Edycja)
  - [Wyświetlenie Produktów](#Wyświetlenie-Produktów)
  - [Wyświetlenie Produktu](#Wyświetlenie-Produktu)
  - [Usunięcie Produktu](#Usunięcie-Produktu)

  ## Rejstracja Użytkownia

Rejstracja użytkownika polega na wypełnieniu formularza składającego się z adresu e-mail oraz hasła. Zatwierdzenie tworzy nowego użytkownika w bazie danych.<br><br>

![Alt text](RedMeObrazy\Rejstracja.png?raw=true "Rejstrtacja")
  


  ## Logowanie Użytkownia

Logowanie użytkownika polega na wypełnieniu formularza składającego się z adresu e-mail oraz hasła.<br><br>

![Alt text](RedMeObrazy\Login.png?raw=true "Logowanie")

# Dodawanie Produktu/Edycja

Dodawanie produktów polega na wypełnieiu formularza: <br><br>

![Alt text](RedMeObrazy\Dodaj.png?raw=true "Dodawanie")

Jeśli wszystkie podane dane są prawidłowe produkt zostaje dodany do bazy danych.<br>
Edycja produktu wygląda prawie identycznie. Jedyną różnica to formularz, który jest wypełniony aktualnymi danymi produktu.

# Wyświetlenie Produktów

Jest to strona startowa. Urzytkownik może z przeglądać listę dostępnch produktów.

![Alt text](RedMeObrazy\Produkty.png?raw=true "Produkty")

# Wyświetlenie Produktu

Wyświetlenie szczegółów danego produktu.

![Alt text](RedMeObrazy\Produkt.png?raw=true "Produkt")


# Usunięcie Produktu

Przyciśnięcie przycisku "Delete" powoduje usunięcie produktu.

![Alt text](RedMeObrazy\Edycja.png?raw=true "Usuń")


