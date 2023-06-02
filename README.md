# 8d-Lattice-DAG-
8D Lattice DAG

#include <iostream>
#include <vector>
#include <random>
#include <bitset>
#include <sstream>
#include <iomanip>
#include <chrono>
#include <thread>
#include <unordered_map>

struct LatticeSymbol {
    unsigned int symbol;
    std::vector<std::string> colors;
    std::bitset<512> complexity;
    std::vector<unsigned int> children; // Stores the indices of child symbols
};

std::vector<LatticeSymbol> createLattice(
    unsigned int width, unsigned int height, unsigned int depth, unsigned int time, unsigned int energy,
    unsigned int dimension7, unsigned int dimension8) {
    
    std::random_device rd;
    std::mt19937 gen(rd());
    std::uniform_int_distribution<unsigned int> distribution(0, 1114111);

    std::vector<LatticeSymbol> lattice(width * height * depth * time * energy * dimension7 * dimension8);

    unsigned int index = 0;
    for (unsigned int i = 0; i < width; i++) {
        for (unsigned int j = 0; j < height; j++) {
            for (unsigned int k = 0; k < depth; k++) {
                for (unsigned int l = 0; l < time; l++) {
                    for (unsigned int m = 0; m < energy; m++) {
                        for (unsigned int n = 0; n < dimension7; n++) {
                            for (unsigned int o = 0; o < dimension8; o++) {
                                unsigned int symbol = distribution(gen);
                                unsigned int numColors = gen() % 10 + 1;
                                std::vector<std::string> colors(numColors);
                                for (unsigned int c = 0; c < numColors; c++) {
                                    colors[c] = "Color" + std::to_string(c + 1);
                                }
                                std::bitset<512> complexity;
                                for (unsigned int b = 0; b < 512; b++) {
                                    complexity[b] = gen() % 2;
                                }

                                lattice[index] = { symbol, colors, complexity, {} };
                                index++;
                            }
                        }
                    }
                }
            }
        }
    }

    return lattice;
}

std::string encryptMessage(const std::string& message,
    const std::vector<LatticeSymbol>& lattice,
    const std::string& encryptionKey, unsigned int numRounds) {
    
    std::vector<unsigned char> encryptedData;
    std::vector<unsigned char> key(encryptionKey.begin(), encryptionKey.end());

    std::random_device rd;
    std::mt19937 gen(rd());
    std::uniform_int_distribution<int> delayDistribution(100, 1000);

    std::unordered_map<unsigned int, const LatticeSymbol*> symbolMap;

    // Build symbol map for easy access using symbol indices
    for (const LatticeSymbol& symbol : lattice) {
        symbolMap[symbol.symbol] = &symbol;
    }

    for (unsigned int round = 0; round < numRounds; round++) {
        for (char c : message) {
            unsigned int symbolIndex = c % lattice.size();
            const LatticeSymbol& latticeSymbol = lattice[symbolIndex];
            std::bitset<512> keyBits = latticeSymbol.complexity;

            unsigned int maxSymbol = latticeSymbol.symbol;
            unsigned int maxComplexity = keyBits.count();
            for (const auto& kvp : symbolMap) {
                unsigned int complexity = kvp.second->complexity.count();
                if (complexity > maxComplexity) {
                    maxSymbol = kvp.first;
                    maxComplexity = complexity;
                }
            }

            std::vector<unsigned long long> keys(8);
            for (unsigned int j = 0; j < 8; j++) {
                std::bitset<64> subKey;
                for (unsigned int b = 0; b < 64; b++) {
                    subKey[b] = keyBits[j * 64 + b];
                }
                keys[j] = subKey.to_ullong();
            }

            std::bitset<64> symbolBits(maxSymbol);
            for (unsigned int i = 0; i < 8; i++) {
                symbolBits ^= std::bitset<64>(keys[i]);
            }

            unsigned long long encryptedSymbol = symbolBits.to_ullong();
            for (unsigned int i = 0; i < 8; i++) {
                std::bitset<64> subKey(keys[i]);
                subKey ^= std::bitset<64>(encryptedSymbol);
                keys[i] = subKey.to_ullong();
            }

            std::vector<unsigned char> encryptedSymbolBytes(8);
            for (unsigned int i = 0; i < 8; i++) {
                encryptedSymbolBytes[i] = (keys[i] >> 56) & 0xFF;
            }

            encryptedData.insert(encryptedData.end(), encryptedSymbolBytes.begin(), encryptedSymbolBytes.end());

            std::this_thread::sleep_for(std::chrono::milliseconds(delayDistribution(gen)));
        }
    }

    std::stringstream ss;
    for (unsigned char byte : encryptedData) {
        ss << std::hex << std::setw(2) << std::setfill('0') << static_cast<int>(byte);
    }

    return ss.str();
}

int main() {
    unsigned int width = ;
    unsigned int height = 10;
    unsigned int depth = 10;
    unsigned int time = 10;
    unsigned int energy = 10;
    unsigned int dimension7 = 10;
    unsigned int dimension8 = 10;

    std::vector<LatticeSymbol> lattice = createLattice(
        width, height, depth, time, energy, dimension7, dimension8);

    std::string message = "Hello, World!";
    std::string encryptionKey = "MySecretKey";
    unsigned int numRounds = 3;

    std::string encryptedMessage = encryptMessage(message, lattice, encryptionKey, numRounds);
    std::cout << "Encrypted Message: " << encryptedMessage << std::endl;

    return 0;
}
