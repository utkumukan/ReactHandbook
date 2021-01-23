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