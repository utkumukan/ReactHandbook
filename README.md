# React Handbook
Herkese merhaba. Bu çalışmada React ile ilgili birkaç konudan senaryolar üzerinde ilerleyerek bahsetmeye çalışıcam. Bütün örneklerin linklerini code bloğunun başlığına etiketledim. Paylaşılan code bloğunun başlığına tıklayarak ilgili linkten detaylı inceleyebilirsiniz.

## Nelerden bahsedicem?
 * bir componentin mount (first render) ve update (every re-render) durumları ve bağımlılıkları
 * hook nedir, kuralları nelerdir?
 * useEffect nedir, ne işe yarar?
 * useEffect kullanımları
 * birkaç örnekle useRef kullanımı
 * parent to child ve child to parent iletişimleri
 * kendi hook'umuzu nasıl yazarız?

## Bir componentin mount (first render) ve update (every re-render) durumları ve bağımlılıkları

 * Bir componentin mount edilmesi durumu hepimizin de bildiği gibi ilk kez çağırılması durumudur. Eğer basit bir componentimiz varsa, mount edilme durumunda çok birşey yapmamıza gerek yoktur. Fakat kompleks bir componente sahipsek, mesela API çağrıları yapan, bu durumda useEffect, useRef gibi hook'lara ihtiyaç duyarız. Bu konuyu ilerleyen senaryolarda useEffect'le birlikte işliycem.

 * Update durumu ise bildiğimiz gibi componentimizin tekrar render edilmesi durumudur. **Bu durumun gerçekleşmesi için ya state'imiz güncellenmiş olmalı ya da props'umuz.** Bu konu da aslında incelenmesi gereken farklı açılara sahip olduğu için ilerleyen senaryolarda işlenecek fakat basit bir örnek üzerinden temel haline bakalım.

    <a href="https://codesandbox.io/s/react-ornek-1-mb5bx" target="_blank"><i>React-Ornek-1</i></a>
    
    ```javascript
    const Child = (props) => {
        const { year } = props;

        return (
            <div>
                <h2>{year}</h2>
            </div>
        );
    };

    const App = () => {
        const [year, setYear] = useState("2020");

        const handleClick = () => {
            setYear("2021");
        };

        return (
            <div className="App" onClick={handleClick}>
                <h1>Gelecek Varlık</h1>
                <Child year={year} />
                <button onClick={handleClick}>Yeni Yılı Getir</button>
            </div>
        );
    };
    ```

    Örnekte de görüldüğü üzere App componentinin state durumunu yönettiği ve bu state'i Child component'i ile paylaştığı bu senaryomuzda butona tıklanması sonucu year state'i güncelleniyor. **State** güncellendiği için önce App component kendini re-render ediyor, App componentinin state'i Child componentinin **props**'u olduğu için haliyle Child componenti de props değşikliğinden dolayı re-render oluyor.

## hook nedir, kuralları nelerdir?

* hook fonksiyon gibi davranan fakat react'taki state ve lifecycle durumlarını da kullanabildiğimiz yapılardır. React'ın bize sunmuş olduğu bazı hook'lar useState, useEffect, useRef vs. şeklindedir. Bu hook'ları kullanarak component'imiz ve kendi yazmış olduğumuz hook'lar içerisinde ihtiyacımız olan aksiyonları sağlayabiliriz.

    Peki bu hook'ların kuralları nelerdir?
    * functional component içerisinde kullanılabilirler,
    * kendi yazmış olduğumuz hook içerisinde kullanılabilirler,
    * useEffect hook'u if içerisine alınamaz,
    * component render edilmeden, güncellenmiş olan useState hook'u yeni değeri ile birlikte kullanılamaz.

    Son maddeye örnek vermek gerekirse 
    ```javascript
    const [amount, setAmount] = useState(100);

    const handleClick = () => {
        const expectedValue = 150;
        setAmount(expectedValue);
        setAmount(amount - 50);
    };
    ```

    Burada ``expectedValue`` değerini amount'a set etmiş olsak da component henüz render olmadığı için amount 150 değerini alamamıştır dolayısıyla ``setAmount(amount - 50)`` ifadesindeki amount değeri hala 100'dür ve handleClick fonksiyonu sonlandıktan sonra amount'un güncel hali 50 olacaktır. 

## useEffect nedir, ne işe yarar?

* useEffect component'imizin render durumlarında (mount, update, only update vs.) veya bu durumları tetikleyen değişkenlerin (state, props vs.) güncellenmesi durumunda çalışan bir hook'tur. Bu sayede component'imizde meydana gelen çeşitli senaryolarda çeşitli aksiyonlar alabiliriz, örneğin BSN kodu değiştiğinde Tahsilat ekranındaki herhangi bir componentin kendini güncellemesi işlemini useEffect sayesinde yaparız.

## useEffect'i her render'da (mount ve update) kullanmak

 * useEffect'in çalışma bağımlılıklarını belirlemek için arrow function'ımızdan sonra köşeli parantez içerisinde bağımlılıkları parametre olarak geçeriz. Fakat bu kullanım component'imizin kullanım durumuna göre değişiklik gösterebilir. Eğer ki component'imizin mount ve update durumlarında useEffect'i çalıştırmasını istiyorsak bağımlılıkları kullanmamıza gerek yoktur.

    Örneğimizi inceleyelim;

    <a href="https://codesandbox.io/s/react-ornek-2-gr4iv" target="_blank"><i>React-Ornek-2</i></a>
 
    ```javascript
    const Child = (props) => {
        const { count } = props;

        useEffect(() => {
            console.log(
            "herhangi bir bağımlılığım olmadığı için her render'da çalışırım"
            );
        });

        return (
            <div>
                <h2>{count}</h2>
            </div>
        );
    };

    const App = () => {
        const [count, setCount] = useState(0);

        const handleMinus = () => {
            setCount(count - 1);
        };

        const handlePlus = () => {
            setCount(count + 1);
        };

        return (
            <div className="App">
                <button onClick={handleMinus}>-</button>
                <Child count={count} />
                <button onClick={handlePlus}>+</button>
            </div>
        );
    };
    ```

    Görüldüğü üzere herhangi bir bağımlılığımız yok, örnek incelendiğinde component mount olduğu durumda console'a ilgili mesaj yazılıyor ve daha sonra yapılan her ekleme ve çıkarma işlemi sonrasında da ilgili mesajı console'da görebiliyoruz. Sonuç olarak useEffect'e bağımlılık verilmemişse aslında bağımlılığı componentin **state**'i ya da **props**'u oluyor. Bu örnekte props'umuz değiştiği için her değişimde useEffect çalışıyor. Eğer ki Child componentinin bir state'i olsaydı o state güncellendiğinde de useEffect çalışırdı.

## useEffect'i sadece mount durumunda kullanmak

 * Eğer ki useEffect'in sadece component'imizin çağırıldığı durumda çalışmasını istiyorsak bağımlılık array'imizi boş bırakmamız gerekir.

    Örneğimizi inceleyelim;
    
    <a href="https://codesandbox.io/s/react-ornek-3-vk7tw" target="_blank"><i>React-Ornek-3</i></a>
    ```javascript
    const Child = (props) => {
        const { count } = props;

        useEffect(() => {
            console.log(
            "bağımlılık array'im boş olduğu için sadece mount durumunda çalışırım"
            );
        }, []);

        return (
            <div>
                <h2>{count}</h2>
            </div>
        );
    };

    const App = () => {
        const [count, setCount] = useState(0);

        const handleMinus = () => {
            setCount(count - 1);
        };

        const handlePlus = () => {
            setCount(count + 1);
        };

        return (
            <div className="App">
                <button onClick={handleMinus}>-</button>
                <Child count={count} />
                <button onClick={handlePlus}>+</button>
            </div>
        );
    };
    ```
    Örnekte de görüldüğü gibi bağımlılık arrayi boş olduğu için useEffect sadece mount durumunda çalışıyor. Fakat dikkat ettiğiniz üzere useEffect'in update durumlarında çalışmıyor olması componentin update olmasına engel değil, her ekleme ve çıkarma işleminde count değerimiz güncelleniyor.

## useEffect'i update durumunda kullanmak
 * Eğer ki useEffect'i herhangi bir update durumunda kullanmak istiyorsak ilgili değeri bağımlılık array'ine parametre olarak geçmemiz yeterlidir.

    Örneği inceleyelim;

    <a href="https://codesandbox.io/s/react-ornek-4-kjee3" target="_blank"><i>React-Ornek-4</i></a>
    ```javascript
    const Child = (props) => {
        const { count } = props;

        useEffect(() => {
            console.log("count'un mount ve update olduğu durumlarda çalışırım");
        }, [count]);

        return (
            <div>
                <h2>{count}</h2>
            </div>
        );
    };

    const App = () => {
        const [count, setCount] = useState(0);

        const handleMinus = () => {
            setCount(count - 1);
        };

        const handlePlus = () => {
            setCount(count + 1);
        };

        return (
            <div className="App">
                <button onClick={handleMinus}>-</button>
                <Child count={count} />
                <button onClick={handlePlus}>+</button>
            </div>
        );
    };
    ```
    Örneği incelediğimizde mount durumunda useEffect'in çalıştığını görürüz, ardından her update işleminde de count değeri bağımlılık array'inde olduğu için useEffect çalışır.

    Gelin şimdi farklı bir örneği inceleyelim, bu örnekte Child component'imizde bir state'imiz olsun ve bu state'i güncelleyelim. Bakalım bu durumda useEffect nasıl bir davranış sergileyecek.

    <a href="https://codesandbox.io/s/react-ornek-5-iwf41" target="_blank"><i>React-Ornek-5</i></a>
    ```javascript
    const Child = (props) => {
        const { count } = props;
        const [input, setInput] = useState("hadi bir şeyler yaz buraya");

        useEffect(() => {
            console.log("count'un mount ve update olduğu durumlarda çalışırım");
        }, [count]);

        return (
            <div className="Child">
                <input
                    type="text"
                    value={input}
                    onChange={(e) => setInput(e.target.value)}
                />
                <h2>{count}</h2>
            </div>
        );
    };

    const App = () => {
        const [count, setCount] = useState(0);

        const handleMinus = () => {
            setCount(count - 1);
        };

        const handlePlus = () => {
            setCount(count + 1);
        };

        return (
            <div className="App">
                <button onClick={handleMinus}>-</button>
                <Child count={count} />
                <button onClick={handlePlus}>+</button>
            </div>
        );
    };
    ```
    Bu örneği incelediğimizde görüyoruz ki input state'imizdeki değişimler component'in update olmasına engel değil fakat useEffect bağımlılıklarında sadece count değeri bulunduğu için console'a sadece count değeri değişince mesaj yazıldığını görüyoruz. 

    useEffect ve update durumlarını incelemek için gelin son örneğimize bakalım;

    <a href="https://codesandbox.io/s/react-ornek-6-yu6qm" target="_blank"><i>React-Ornek-6</i></a>
    ```javascript
    const Child = (props) => {
        const { count } = props;
        const [input, setInput] = useState("hadi bir şeyler yaz buraya");

        useEffect(() => {
            console.log("count ve input değerlerinin mount ve update durumlarında çalışırım");
        }, [count, input]);

        return (
            <div className="Child">
                <input
                    type="text"
                    value={input}
                    onChange={(e) => setInput(e.target.value)}
                />
                <h2>{count}</h2>
            </div>
        );
    };

    const App = () => {
        const [count, setCount] = useState(0);

        const handleMinus = () => {
            setCount(count - 1);
        };

        const handlePlus = () => {
            setCount(count + 1);
        };

        return (
            <div className="App">
                <button onClick={handleMinus}>-</button>
                <Child count={count} />
                <button onClick={handlePlus}>+</button>
            </div>
        );
    };
    ```
    Bu örnekte ise useEffect count ve input'a aynı anda bağımlı olduğu için bir tanesinin dahi güncellenmiş olması useEffect'in çalışması için yeterlidir.

    Extra:

    <a href="https://codesandbox.io/s/react-ornek-7-oq587" target="_blank"><i>React-Ornek-7</i></a>
    ```javascript
    useEffect(() => {
        console.log("count değerinin mount ve update durumlarında çalışırım");
    }, [count]);

    useEffect(() => {
        console.log("input değerinin mount ve update durumlarında çalışırım");
    }, [input]);
    ```

    Eğer ki useEffect içerisinde karmaşık bir yapı kurmak istemiyorsanız herbir bağımlılık için ayrı ayrı useEffect oluşturabilirsiniz. mount durumunda her iki useEffect de çalışır, sonraki güncelemelerde ise hangi bağımlılıkta değişim varsa sadece o bağımlılığın useEffect'i çalışır.

## useRef yardımıyla useEffect'i sadece update durumunda kullanmak

* Bundan önceki örneklerimizde update senaryolarını irdelerken mount durumunda da useEffect'in çalıştığını görmüştük. Fakat bu her zaman isteyeceğimiz bir durum olmayabilir. Bu senaryoda useRef hook'undan faydalanarak useEffect'i sadece update durumunda çalıştırıcaz.
    
    Örneğimizi inceleyeyim;

    <a href="https://codesandbox.io/s/react-ornek-8-kl9o5" target="_blank"><i>React-Ornek-8</i></a>
    ```javascript
    const Child = (props) => {
        const { count } = props;
        const didMount = useRef(false);

        useEffect(() => {
            if (didMount.current) {
                console.log("count değerinin update olduğu durumlarda çalışırım");
            } else {
                didMount.current = true;
            }
        }, [count]);

        return (
            <div className="Child">
                <h2>{count}</h2>
            </div>
        );
    };

    const App = () => {
        const [count, setCount] = useState(0);

        const handleMinus = () => {
            setCount(count - 1);
        };

        const handlePlus = () => {
            setCount(count + 1);
        };

        return (
            <div className="App">
                <button onClick={handleMinus}>-</button>
                <Child count={count} />
                <button onClick={handlePlus}>+</button>
            </div>
        );
    };
    ```
    Örneği incelediğimizde görüyoruz ki useEffect'in count değerine bağlı çalışması durumunda ref yardımı ile if kontrolü sağlayarak update durumlarına özel aksiyon sağlayabiliyoruz, mount durumunda ise inaktif bırakabiliyoruz. İlk değerini false atadığımız useRef'in current değeri, count prop'unun mount olması sonucunda çalışan useEffect içerisinde basit bir if else kontrolü sayesinde current'ına true değeri ataması yapılıyor. Artık değer true olduğu için update işlemlerinde else bloğuna girmeyecek ve istediğimiz aksiyonu sağlayabileceğiz. 
    
    Tahmin ettiğiniz gibi bu senaryoda mount durumu için ayrı update durumu için ayrı ya da her count değişimi için bir aksiyon almak istersek else durumuna istediğimiz eklemeyi yapabiliriz.

    Örnek vermek gerekirse; 

    ```javascript
    useEffect(()=> {
        console.log("count değerinin RENDER olduğu durumlarda çalışırım")
        if (didMount.current) {
            console.log("count değerinin UPDATE olduğu durumlarda çalışırım");
        } else {
            console.log("count değerinin MOUNT olduğu durumda çalışırım");
            didMount.current = true;
        }
    }, [count]);
    ```

## useRef yardımıyla useEffect'i yalnızca bir kez kullanmak

* useEffect'i yalnızca bir kez kullandığımız bir senaryomuz vardı fakat o senaryoda bağımlılık array'ini boş geçerek yalnızca mount durumunda çalıştırıyorduk useEffect'imizi. Bu senaryomuzda ise bağımlılığımız'ın özel bir durumunu kontrol edip ilgili update'i yakalayınca useEffect'imizi kullanıcaz.

    Örneğimizi inceleyeyim;

    <a href="https://codesandbox.io/s/react-ornek-9-kdon8" target="_blank"><i>React-Ornek-9</i></a>
    ```javascript
    const Child = (props) => {
        const { count } = props;
        const calledOnce = useRef(false);

        useEffect(() => {
            if (calledOnce.current) {
                return;
            }

            if (count === 5) {
                console.log("count değeri 5 olduğunda çalışırım");
                calledOnce.current = true;
            }
        }, [count]);

        return (
            <div className="Child">
                <h2>{count}</h2>
            </div>
        );
    };

    const App = () => {
        const [count, setCount] = useState(0);

        const handleMinus = () => {
            setCount(count - 1);
        };

        const handlePlus = () => {
            setCount(count + 1);
        };

        return (
            <div className="App">
                <button onClick={handleMinus}>-</button>
                <Child count={count} />
                <button onClick={handlePlus}>+</button>
            </div>
        );
    };
    ```

    Örneğimizde görüyoruz ki useEffect'in count değerine bağlı çalışması durumunda ref yardımı ile if kontrolü yaparak count değeri 5 olmadığı sürece ilgili aksiyon sağlanmıyor ve ardından calledOnce useRef'imizin current değerini true yaptığımız için count değerini bir daha 5 yapsak dahi ilgili aksiyonu alamayız. Bunu önlemek için ````calledOnce.current = true```` kullanımından kaçınabiliriz. Böyle bir kullanımda istediğimiz aksiyonu birden fazla kez sağlamış oluruz fakat sadece 5 değerine eşit olduğunda.

    Bu başlık için count örneği tabii ki mantıksız fakat gerçek bir projede faydalanmak istersek boolean bir bağımlılığımız varsa daha kullanışlı olabilir.

    Örnek vermek gerekirse;

    ```javascript
    useEffect(() => {
        if (calledOnce.current) {
            return;
        }

        if (sgkFlag === true) {
            getSocialInsurences();
        }
    }, [sgkFlag]);
    ```

## parent to child (p->c) & child to parent (c->p) ilişkileri

* **p->c** ilişkisini kurmak için kullandığımız component'e prop geçmemiz yeterli ve bunu sık sık yapıyoruz. Örneğin;

    ```javascript
    const [code] = useState('BSN0000000');

    return <Customer code={code} />;
    ```

    Bu kullanımda Customer component'ini çağırdığımız yer *Parent* component'imiz oluyor ve Customer component'i de *Child* component olmuş oluyor. *Parent*'tan *Child*'a prop gönderdiğimiz için de **p->c** ilişkisini kurmuş oluyoruz.

* Aslında **c->p** ilişkisi de aynı mantığı taşıyor fakat burda ilişkinin tersine dönüyor olmasının nedeni *Parent*'tan *Child*'a geçmiş olduğumuz prop'un bir function olması. prop'ta gönderilen function'ın tanımlı olduğu yer *Parent* component'i olduğu için, *Child* component'i bu function'ı kullandığında *Parent* component'ine geri dönüş yapmış oluyor. Bunun da bir örneğine bakalım; 

    <a href="https://codesandbox.io/s/react-ornek-10-rlekj" target="_blank"><i>React-Ornek-10</i></a>
    ```javascript
    const App = () => {
        const [count, setCount] = useState(0);

        const handleChange = (item) => {
            setCount(item);
        };

        return (
            <div className="App">
                <Child onChange={handleChange} />
                {count}
            </div>
        );
    };

    const Child = (props) => {
        const { onChange } = props;

        const handleClick = () => {
            onChange(Math.floor(Math.random() * (100 - 0) + 0));
        };

        return <button onClick={handleClick}>Rasgele Sayı</button>;
    };
    ```

    Örneği incelediğimizde görüyoruz ki Parent component'inde count state'imiz var ve bu component içinde count state'ini set ediyoruz fakat bu yeni değere karar veren component Child component'i. Parent'tan Child'a geçtiğimiz ``onChange`` prop'u bir function ve Child component'i bu function'a parametre geçerek Parent component'indeki ``onChange``'in değeri/tanımı olan ``handleChange``'e bu parametreyi göndermiş oluyor. Parent component'i de Child component'inden gelen bu değeri kullanarak kendi bünyesinde bulunan count state'ini update ediyor. Bu şekilde **c->p** ilişkisini kurmuş olduk.

* Gelin şimdiki örneğimize bakalım. Bu örnekte **p->c** ve **c->p** ilişkilerini useEffect'le birlikte kullanarak zincirleme bir yapı oluşturdum.

    <a href="https://codesandbox.io/s/react-ornek-11-jgc3w" target="_blank"><i>React-Ornek-11</i></a>
    ```javascript
    const App = () => {
        const [companyName, setCompanyName] = useState("Gelecek Varlık");

        const handleChange = (item) => {
            setCompanyName(item);
        };

        return (
            <div className="App">
                <fieldset>
                    <legend>App</legend>
                    <h1>{`${companyName} Ailesine Hoş Geldiniz`}</h1>
                    <Parent data={companyName} onChange={handleChange} />
                </fieldset>
            </div>
        );
    };

    const Parent = (props) => {
        const { data, onChange } = props;
        const [companyName, setCompanyName] = useState("");

        const handleChange = (item) => {
            onChange(item);
        };

        useEffect(() => {
            setCompanyName(data);
        }, [data]);

        return (
            <div>
                <fieldset>
                    <legend>Parent</legend>
                    <p>{`${companyName} Adresi: Kağıthane`}</p>
                    <Child1 data={companyName} />
                    <Child2 data={companyName} onChange={handleChange} />
                </fieldset>
            </div>
        );
    };

    const Child1 = (props) => {
        const { data } = props;
        const [companyName, setCompanyName] = useState(data);

        useEffect(() => {
            setCompanyName(data);
        }, [data]);

        return (
            <div>
                <fieldset>
                    <legend>Child1</legend>
                    <div>{`${companyName} Telefon Numarası: 02124444444`}</div>
                </fieldset>
            </div>
        );
    };

    const Child2 = (props) => {
        const { data, onChange } = props;
        const [state] = useState({
            name1: "Gelecek Varlık",
            name2: "Güven Varlık"
        });

        const handleChange = (e) => {
            onChange(e.target.value);
        };

        return (
            <div>
                <fieldset>
                    <legend>Child2</legend>
                    <div>
                        <label>{state.name1}: </label>
                        <input
                            type="radio"
                            value={state.name1}
                            onChange={handleChange}
                            checked={data === state.name1}
                        />
                    </div>
                    <div>
                        <label>{state.name2}: </label>
                        <input
                            type="radio"
                            value={state.name2}
                            onChange={handleChange}
                            checked={data === state.name2}
                        />
                    </div>
                </fieldset>
            </div>
        );
    };
    ```

    Bu örnekte dört tane component'imiz bulunmakta ve hiyerarşi **App->Parent->Child1==Child2** şeklinde fakat konunun başında da bahsettiğim gibi bir üst component'ten bir altındaki component'e gönderilen function prop güncellenecek paramtreyi bir üstündeki component'e aktarıyor ve useEffect başlıklarında da bahsettiğim gibi gelen prop'un değişimini useEffect'e bağlayıp ilgili state'in set edilmesi işlemini gerçekleştiriyoruz yani bu işlem gerçekleşirken **App->Parent->Child1==Child2** ilişkisi tam tersi davranış sergiliyor. App component'i kendini güncelleyince ise ilişki **p->c** ilişkisine geri dönüyor.

## kendi hook'umuzu nasıl yazarız?

* kendi hook'umuzu nasıl yazarızdan önce neden yazarız sorusuna değinmekte fayda var. Eğer bir component'te ya da bir projede bir işlemi birden fazla kez kullanıyorsak bunu bir hook yardımıyla daha sade bir kullanım haline getirebiliriz. 

    Örnek vermek gerekirse bir projede belli başlı bilgileri localStorage'a yazarız ve yeri geldiğinde bu bilgiyi localStorage'dan isteriz. Bunu bir çok yerde yaptığımızda gereksiz kod kalabalığı yaratmış oluruz, bunun yerine localStorage'da yazma ve okuma işlemi yapan custom bir hook oluşturursak hem daha okunaklı hem de maintainable bir kodumuz olmuş olur.

* hook'lar aslında function gibi davranırlar peki neden function demiyoruz da hook diyoruz? Çünkü hook'ları yazarken (en başta da bahsettiğim gibi) react'ın functional component'lere sunmuş olduğu hook'ları (useState, useEffect, useRef vs.) kullanırız ve hook'lar sadece functional component'lerde ve kendi yazacağımız hook'larda kullanılabilirler, bu yüzden hook'ların function olduklarını söyleyemeyiz.

* Bu başlık için iki tane hook (useLocalStorage ve useRandomNumber) oluşturdum gelin şimdi onları inceleyelim.

    <a href="https://codesandbox.io/s/uselocalstoragehook-54gno" target="_blank"><i>React-Ornek-12</i></a>
    ```javascript
    const useLocalStorageState = (key, value) => {
        const [state, setState] = useState(() => {
            try {
                return JSON.parse(window.localStorage.getItem(key) || value);
            } catch (e) {
                console.log(e);
                return value;
            }
        });

        useEffect(() => {
            window.localStorage.setItem(key, state);
        }, [key, state]);

        return [state, setState];
    };

    const App = () => {
        const [count, setCount] = useLocalStorageState("localStorage-count", 0);

        return (
            <div className="App">
                <button onClick={() => setCount(count + 1)}>{count}</button>
            </div>
        );
    };
    ```

    *useState* ve *useEffect* kullanarak oluşturduğum *useLocalStorage* hook'u key ve value şeklinde iki parametre alıyor ve return ettiği değerler useState'in state ve setState durumları. Bu nedenledir ki kullanmak istediğimiz yerde tanımlarken aynı useState tanımlar gibi tanımlama yapıyoruz (destructring). Diğer işlemlerse aslında bildiğimiz localStorage'dan okuma ve yazma işlemleri. Bunu yaparken önce hook'umuzda bir state tanımlıyoruz ve bu state'in initial değeri parametre geçilen key değerinden yapılan okumadan (eğer hata yoksa) geliyor. Eğer ilgili key yoksa o an geçilmiş olan value değeri state'e atanıyor. Daha sonra state değişiminden dolayı re-render olan hook'umuz useEffect içerisinde yeni state değerini localStorage'a yazıyor. 
    
    Bu şekilde basit bir hook yazarak localStorage işlemlerimizi gerçekleştirmiş olduk.

    <a href="https://codesandbox.io/s/userandomnumber-fhuj9" target="_blank"><i>React-Ornek-13</i></a>
    ```javascript
    function useRandomNumber(range) {
        const [state, setState] = useState();

        function getRandomNumber(min, max) {
            min = Math.ceil(min);
            max = Math.floor(max);
            return Math.floor(Math.random() * (max - min) + min);
        }

        useEffect(() => {
            switch (range) {
            case "0-25":
                setState(getRandomNumber(0, 25));
                break;
            case "25-50":
                setState(getRandomNumber(25, 50));
                break;
            case "50-75":
                setState(getRandomNumber(50, 75));
                break;
            case "75-100":
                setState(getRandomNumber(75, 100));
                break;
            default:
                setState(0);
            }
        }, [range]);

        return state;
    }

    const App = () => {
        const [range, setRange] = useState("");
        const value = useRandomNumber(range);

        const handleChange = (e) => {
            setRange(e.target.value);
        };

        return (
            <div className="App">
                <h1>Rastgele Sayı</h1>
                <div className="Child">
                    <div className="Input">
                        <label>0 - 25</label>
                        <input
                            type="radio"
                            value="0-25"
                            checked={range === "0-25"}
                            onChange={handleChange}
                        />
                    </div>
                    <div className="Input">
                        <label>25 - 50</label>
                        <input
                            type="radio"
                            value="25-50"
                            checked={range === "25-50"}
                            onChange={handleChange}
                        />
                    </div>
                    <div className="Input">
                        <label>50 - 75</label>
                        <input
                            type="radio"
                            value="50-75"
                            checked={range === "50-75"}
                            onChange={handleChange}
                        />
                    </div>
                    <div className="Input">
                        <label>75 - 100</label>
                        <input
                            type="radio"
                            value="75-100"
                            checked={range === "75-100"}
                            onChange={handleChange}
                        />
                    </div>
                </div>
                <h2>{value}</h2>
            </div>
        );
    };
    ```
    *useState* ve *useEffect* kullanarak oluşturduğum *useRandomNumber* hook'u basit bir işlemle kendisine verilen aralıkta bir sayı generate edip return ediyor.

    App component'inde range state'imiz var ve radio button'lara tıkladığımızda ilgili değer set ediliyor. State set edildiği için component re-render oluyor ve *useRandomNumber*'a yeni değer gönderilmiş oluyor. *useRandomNumber* ise gelen range bilgisine göre *useEffect* içerisinde return edilecek state'i set ediyor ve return işlemi ile aksiyonunu tamamlıyor, bu şekilde de hook'umuzun kullanıldığı component'te hook tarafından generate edilmiş sayıyı görebiliyoruz.


### Umarım faydalı olmuştur. İyi çalışmalar herkese :)
    
