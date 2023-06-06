# Trifecta

This is a software to benchmark Trifecta, a faster high-throughput three-party computation protocol. This is as part of the submission procedure for PoPETS 23 where the theoretical work will be presented. 

The implementation here is a fork of [MP-SPDZ](https://github.com/data61/MP-SPDZ/tree/master), a general toolkit to prototype and benchmark various multi-party computation (MPC) protocol. The vanilla MP-SPDZ doesn't support computations on multi-fan-in AND gates, required by protocols such as Trifecta. Therefore, we make multiple improvements in the code to enable this feature:

1. We modify the built-in MP-SPDZ compiler to accept representations of Boolean circuits with multi-fan-in AND gates in the source code. The compiler thus generates a bytecode that is augmented to  include multi-fan-in instructions. In our use-case, we extend the [Bristol Fashion](https://homes.esat.kuleuven.be/~nsmart/MPC/) MPC circuits with multi-fan-in gates which are then translated by the compiler.
2. Subsequently, the virtual machine processor is enhanced to parse and evaluate the new multi-fan-in instructions in the bytecode accordignly in a backward-compatible manner. 
3. We implement Trifecta on top this upgraded toolchain for experimental evalution of our protocol and test the accuracy of our build. 

We note that the changes introduced here are not specific to Trifecta and the underlying framework can be used to benchmark any protocol that supports computation of multi-fan-in AND gates [].

# <a name="compilation"></a> Compilation 

Please follow the instructions in the [MP-SPDZ](https://github.com/data61/MP-SPDZ/tree/master) to install all the requirements. On an Ubuntu machine, the following one-liner should be enough:

```
sudo apt-get install automake build-essential clang cmake git libboost-dev libboost-thread-dev libgmp-dev libntl-dev libsodium-dev libssl-dev libtool python3 yasm m4
```

Since this is a fork of an older version of MP-SPDZ, there is a nuance additional configuration step required to run the software. We rely on the MPIR Library for integer operations. The MPIR folder contains the latest release of the library provided here for convinience. To complete the installation 

```
cd mpir

./configure --enable-cxx 
make

make check && make install
ldconfig
```

Run the following command to compile the binary for the Trifecta machine in the main directory

```
make multi-replicated-bin-party.x
```

You can speed up this last step by adding ``` -j8 ``` flag to the previous command. 



## Docker

The command below creates a docker image installing all the dependencies and generates player certificates and keys.

```
docker build --tag mpspdz:trifecta .
```


# Running Computations

To run a computation, you first need to compile the source code. For example ```rc_adder.mpc ``` program is as follows

```
from circuit import Circuit
sb64 = sbits.get_type(64)
adder = Circuit('rc_adder_64')
a = sbitvec(sb64.get_input_from(0))
b = sbitvec(sb64.get_input_from(0))
print_ln('%s', adder(a, b).elements()[0].reveal())
```

This will take two 64-bit numbers from party 0 as input, adds them using a ripple-carry adder, printing the result of the computation. To compile the program run

```
./compile.py -B 64 rc_adder
```

where ``` -B ``` indicates the bit-length of the input. Then to run the program on the same machine, run:

``` 
./multi-replicated-bin-party -I -p 0 rc_adder
```

``` 
./multi-replicated-bin-party -I -p 1 rc_adder
```

``` 
./multi-replicated-bin-party -I -p 2 rc_adder
```

in separate terminals. To run on separate machines,  the same command on each host with ``` --ip-file-name Player-Data/ip-file ``` where ip-file is as described in [Compilation](#compilation). The ```-I``` flag prompts the user for inputs in an interactive mode. Omit this flag to read inputs from ``` Player-Data/Input-P<party number>-0 ``` in text format. 

We have provided our depth-optimized circuits using multi-fan-in AND gates for multiple functionalities in ``` Programs/Circuits ```. Follow the same instructions to compile and run a new computation by simply substituting the circuit in the source code with the desired one and adjusting the input bit-length. You can find more examples and template programs in ``` Programs/Source ```. 

