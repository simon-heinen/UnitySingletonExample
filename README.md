# UnitySingletonExample - A Singleton Pattern without static fields

The core idea is to use GameObjects to represent singletons which has multiple advantages:

* No need to use static fields
* No need to modify existing MonoBehaviours / Components to use them as Singletons
* Easy to reset (just destroy the Singletons GameObject)
* Easy to inject mocks (just initialize it with the mock before using it)
* Inspection and configuration using normal Unity editor and can happen already on editor time

![A singleton generated on runtime](https://i.imgur.com/wKvdrg7.png)

Test.cs:

    public class Test : MonoBehaviour {
        void Start() {
            ExampleSingleton singleton = ExampleSingleton.instance;
            Assert.IsNotNull(singleton); // automatic initialization on first usage
            Assert.AreEqual("abc", singleton.myVar1);
            singleton.myVar1 = "123";
            // multiple calls to instance() return the same object:
            Assert.AreEqual(singleton, ExampleSingleton.instance); 
            Assert.AreEqual("123", ExampleSingleton.instance.myVar1);
        }
    }

ExampleSingleton.cs

    public class ExampleSingleton : MonoBehaviour {
        public static ExampleSingleton instance { get { return Singleton.get<ExampleSingleton>(); } }
        public string myVar1 = "abc";
        public void Start() { Assert.AreEqual(this, instance, "Singleton more than once in scene"); } 
    }

    /// <summary> Helper that turns any MonBehaviour or other Component into a Singleton </summary>
    public static class Singleton {
        public static T get<T>() where T : Component {
            return GetOrAddGo("Singletons").GetOrAddChild("" + typeof(T)).GetOrAddComponent<T>();
        }
        private static GameObject GetOrAddGo(string goName) {
            var go = GameObject.Find(goName);
            if (go == null) { return new GameObject(goName); }
            return go;
        }
    }

    public static class GameObjectExtensionMethods { 
        public static GameObject GetOrAddChild(this GameObject parentGo, string childName) {
            var childGo = parentGo.transform.FindChild(childName);
            if (childGo != null) { return childGo.gameObject; } // child found, return it
            var newChild = new GameObject(childName);        // no child found, create it
            newChild.transform.SetParent(parentGo.transform, false); // add it to parent
            return newChild;
        }

        public static T GetOrAddComponent<T>(this GameObject parentGo) where T : Component {
            var comp = parentGo.GetComponent<T>();
            if (comp == null) { return parentGo.AddComponent<T>(); }
            return comp;
        }
    }
