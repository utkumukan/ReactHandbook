# React Handbook
Herkese merhaba. Bu çalışmada React ile ilgili birkaç konudan senaryolar üzerinde ilerleyerek bahsetmeye çalışıcam. Bütün örneklerin linklerini code bloğunun başlığına etiketledim. Paylaşılan code bloğunun başlığına tıklayarak ilgili linkten detaylı inceleyebilirsiniz.

## Nelerden bahsedicem?
 * bir componentin mount (first render) ve update (every re-render) durumları ve bağımlılıkları, 
 * agırlıklı olarak useEffect kullanımı, 
 * birkaç örnekle useRef kullanımı, 
 * child-parent ve parent-child iletişimleri ve
 * kendi hook'umuzu nasıl yazarız.

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

    Örnekte de görüldüğü üzere App componentinin state durumunu yönettiği ve bu state'i Child component'i ile paylaştığı bu senaryomuzda butona tıklanması sonucu year state'i güncelleniyor. **State** güncellendiği için önce App komponent kendini re-render ediyor, App componentinin state'i Child componentinin **props**'u olduğu için haliyle Child componenti de props değşikliğinden dolayı re-render oluyor.

## useEffect'i her durumda (mount ve update) kullanmak

 * useEffect'in çalışma bağımlılıklarını belirlemek için arrow function'ımızdan sonra köşeli parantez içerisinde bağımlılıkları parametre olarak geçeriz. Fakat bu kullanım component'imizin kullanım durumuna göre değişiklik gösterebilir. Eğer ki component'imizin mount ve update durumlarında useEffect'i çalıştırmasını istiyorsak bağımlılıkları kullanmamıza gerek yoktur.

    Örneğimizi inceleyelim;

    <a href="https://codesandbox.io/s/react-ornek-2-gr4iv" target="_blank"><i>React-Ornek-2</i></a>
 
    ```javascript
    const Child = (props) => {
        const { count } = props;

        useEffect(() => {
            console.log(
            "herhangi bir bağımlılığım olmadığı için her durumda çalışırım"
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

## useEffect'i sadece update durumunda kullanmak

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
    Örneği incelediğimizde görüyoruz ki useEffect sadece update durumlarında çalışıyor, mount durumunda inaktif. Bunu sağlayan ise useRef ile component'in mount olup olmadığını basit bir şekilde kontrol edebiliyor olmamız. İlk değerini false atadığımız useRef'in current değeri, count prop'unun mount olması sonucunda çalışan useEffect içerisinde basit bir if else kontrolü sayesinde current'ına true değeri ataması yapılıyor. Artık değer true olduğu için update işlemlerinde else bloğuna girmeyecek ve istediğimiz aksiyonu sağlayabileceğiz. 
    
    Tahmin ettiğiniz gibi bu senaryoda mount durumu için ayrı update durumu için ayrı bir aksiyon almak istersek else durumuna istediğimiz eklemeyi yapabiliriz.

    Örnek vermek gerekirse; 

    ```javascript
    useEffect(()=> {
        if (didMount.current) {
            console.log("count değerinin UPDATE olduğu durumlarda çalışırım");
        } else {
            console.log("count değerinin MOUNT olduğu durumda çalışırım");
            didMount.current = true;
        }
    }, [count]);
    ```

## useEffect'i yalnızca bir kez kullanmak

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

    Örneğimizde görüyoruz ki count değeri 5 olmadığı sürece useEffect'imiz çalışmıyor ve ardından calledOnce useRef'imizin current değerini true yaptığımız için count değerini bir daha 5 yapsak dahi useEffect çalışmaz. Bunu önlemek için ````calledOnce.current = true```` kullanımından kaçınabiliriz. Böyle bir kullanımda useEffect'i birden fazla kez çalıştırmış oluruz fakat sadece 5 değerine eşit olduğunda.

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
    }, [count]);
    ```
