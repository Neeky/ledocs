```python
#!/usr/bin/evn python3

import weakref
import sys

caches = {}


class Person(object):
    def __init__(self, name: str = "tom"):
        """
        """
        self.name = name

    def __del__(self):
        print(f"{self.name} dead.")

    def __str__(self):
        return f"Person instance {self.name} ."


def release_cache(weak_ref_object):
    """
    """
    k = hash(weak_ref_object)
    del caches[k]

    print(caches)
    print(f"弱引用对象 {weak_ref_object} 所引用的对象被释放了 .")


def main():
    """
    当对象被回收后，一并回收缓存中的内容。
    """
    tom = Person('tom')
    jer = Person('jer')
    rtom = weakref.ref(tom, release_cache)
    rjer = weakref.ref(jer, release_cache)

    # 缓存中只保存对象的弱引用
    caches.update({hash(rtom): rtom, hash(rjer): rjer})
    print(caches)
    print('-'*48)

    print("准备删除 tom .")
    tom = None
    print("删除完成")

    print('-'*48)


main()
```