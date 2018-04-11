

# cluster-index
### faiss 라이브러리 설치 및 사용법 및 예제들 백업
### Polysemous Code, Product Quantization

### faiss 빌드 과정

1. makefile.inc 작성 (또는, fiass 제공하는 템플릿 수정)
2. blas 라이브러리 선택/설치 (intel mkl)
3. SWIG 설치
4. Build python interface (CPU version)

```bash
# makefile.inc
cp ${faiss}/example_makefiles/makefile.inc.Linux ${faiss}/makefile.inc

# Intel MKL .bashrc
MKLROOT="/opt/intel/compilers_and_libraries/linux/mkl/"
export LD_LIBRARY_PATH=${MKLROOT}/lib64:${LD_LIBRARY_PATH}
export LD_PRELOAD=${MKLROOT}/lib/intel64/libmkl_core.so:${MKLROOT}/lib/intel64/libmkl_sequential.so

# SWIG install
sudo apt-get install swig  # swig 3.0 설치

# SWIG .bashrc
# python -c "import distutils.sysconfig; print distutils.sysconfig.get_python_inc()"
# python -c "import numpy ; print numpy.get_include()"
PYTHONCFLAGS=-I/usr/include/python2.7/ -I/usr/lib64/python2.7/site-packages/numpy/core/include/

# Build faiss
${faiss}/make -j8

# Build python interface
${faiss}/make py
```

5. CUDA 설치 (옵션 - GPU 가속 라이브러리, g++11 and cublas)
```bash
# download from developer.nvidia.com/cublas
# Check compute Capability from developer.nvidia.com/cuda-gpus
# update makefile.inc gencode 1080-Ti 6.1

# ${faiss}/gpu/Makefile  -Wl,--no-as-needed 옵션 제거 (ld unrecognized -Wl)
BLASLDFLAGS=-L$(MKLROOT)/lib/intel64 -lmkl_intel_ilp64 -lmkl_core -lmkl_gnu_thread -ldl -lpthread
BLASLDFLAGSNVCC= $(BLASLDFLAGS) 
BLASLDFLAGSSONVCC= $(BLASLDFLAGS)

# Build Faiss (GPU)
${faiss}/gpu/make -j8

# Build python interface (GPU)
${faiss}/gpu/make py
```

6. faiss 시스템 전역 설치 (setup.py 작성)
```python
from setuptools import setup, find_packages

setup(
    name='faiss',
    version='0.1',
    include_package_data=True,
    packages=find_packages(),
    description='Faiss library',
)
```

```bash
# 설치
sudo pip install -e .
```

7. faiss 정상 동작 테스트
```python
import faiss, numpy
print(faiss.Kmeans(10, 20).train(numpy.random.rand(1000, 10).astype('float32')))
```

## Library
1. [faiss (polysemous code) by facebook](https://github.com/facebookresearch/faiss)
