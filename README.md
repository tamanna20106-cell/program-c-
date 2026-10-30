#include
#include
#include
#include
#include
#include
using namespace std;

// Abstract class for farming advice
class Advisory {
public:
virtual void giveAdvice() const = 0;
virtual ~Advisory() {}
};

// Base class
class Crop {
protected:
string name, season;
double area, yieldPerAcre;
static int cropCount;

public:
// Default constructor
Crop() : name("Unknown"), season("Unknown"),
area(0), yieldPerAcre(0) {
cropCount++;
}

// Parameterized constructor
Crop(string n, double a, string s, double y)
    : name(n), season(s), area(a), yieldPerAcre(y) {
    if (a <= 0 || y < 0)
        throw invalid_argument("Invalid area or yield.");
    cropCount++;
}

// Virtual destructor
virtual ~Crop() {
    cropCount--;
}

// Calculate total yield
virtual double totalYield() const {
    return area * yieldPerAcre;
}

virtual void display() const {
    cout << fixed << setprecision(2);
    cout << "\nCrop Name: " << name;
    cout << "\nSeason: " << season;
    cout << "\nArea: " << area << " acres";
    cout << "\nYield per Acre: " << yieldPerAcre;
    cout << "\nTotal Yield: " << totalYield() << " units\n";
}

// Operator overloading to compare yields
bool operator>(const Crop& other) const {
    return totalYield() > other.totalYield();
}

// Save crop data to file
virtual void save(ofstream& file) const = 0;

static int getCropCount() {
    return cropCount;
}
};

int Crop::cropCount = 0;

// Derived class for cereal crops
class CerealCrop : public Crop, public Advisory {
string fertilizerType;

public:
// Default constructor
CerealCrop() : Crop(), fertilizerType("Not specified") {}

// Parameterized constructor
CerealCrop(string n, double a, string s, double y,
           string f)
    : Crop(n, a, s, y), fertilizerType(f) {}

void display() const override {
    cout << "\n--- Cereal Crop Details ---";
    Crop::display();
    cout << "Fertilizer Type: " << fertilizerType << "\n";
}

void giveAdvice() const override {
    cout << "Advice: Use balanced fertilizers and "
         << "monitor soil nutrients regularly.\n";
}

void save(ofstream& file) const override {
    file << "Cereal Crop\n"
         << "Name: " << name << "\n"
         << "Season: " << season << "\n"
         << "Area: " << area << " acres\n"
         << "Yield per Acre: " << yieldPerAcre << "\n"
         << "Fertilizer: " << fertilizerType << "\n"
         << "Total Yield: " << totalYield() << " units\n\n";
}
};

// Derived class for vegetable crops
class VegetableCrop : public Crop, public Advisory {
int irrigationFrequency;

public:
// Default constructor
VegetableCrop() : Crop(), irrigationFrequency(1) {}

// Parameterized constructor
VegetableCrop(string n, double a, string s, double y,
              int frequency)
    : Crop(n, a, s, y), irrigationFrequency(frequency) {
    if (frequency <= 0)
        throw invalid_argument("Invalid irrigation frequency.");
}

void display() const override {
    cout << "\n--- Vegetable Crop Details ---";
    Crop::display();
    cout << "Irrigation Frequency: "
         << irrigationFrequency << " times per week\n";
}

void giveAdvice() const override {
    cout << "Advice: Maintain proper soil moisture "
         << "and avoid over-irrigation.\n";
}

void save(ofstream& file) const override {
    file << "Vegetable Crop\n"
         << "Name: " << name << "\n"
         << "Season: " << season << "\n"
         << "Area: " << area << " acres\n"
         << "Yield per Acre: " << yieldPerAcre << "\n"
         << "Irrigation: " << irrigationFrequency
         << " times per week\n"
         << "Total Yield: " << totalYield() << " units\n\n";
}
};

int main() {
try {
string cerealName, cerealSeason, fertilizer;
string vegetableName, vegetableSeason;
double cerealArea, cerealYield;
double vegetableArea, vegetableYield;
int irrigation;

    cout << "SMART FARM MANAGEMENT SYSTEM\n";
    cout << "----------------------------\n";

    cout << "\nEnter cereal crop name: ";
    getline(cin, cerealName);

    cout << "Enter cereal season: ";
    getline(cin, cerealSeason);

    cout << "Enter cereal area in acres: ";
    if (!(cin >> cerealArea) || cerealArea <= 0)
        throw invalid_argument("Area must be positive.");

    cout << "Enter cereal yield per acre: ";
    if (!(cin >> cerealYield) || cerealYield < 0)
        throw invalid_argument("Yield cannot be negative.");

    cin.ignore(numeric_limits<streamsize>::max(), '\n');

    cout << "Enter fertilizer type: ";
    getline(cin, fertilizer);

    cout << "\nEnter vegetable crop name: ";
    getline(cin, vegetableName);

    cout << "Enter vegetable season: ";
    getline(cin, vegetableSeason);

    cout << "Enter vegetable area in acres: ";
    if (!(cin >> vegetableArea) || vegetableArea <= 0)
        throw invalid_argument("Area must be positive.");

    cout << "Enter vegetable yield per acre: ";
    if (!(cin >> vegetableYield) || vegetableYield < 0)
        throw invalid_argument("Yield cannot be negative.");

    cout << "Enter irrigation frequency per week: ";
    if (!(cin >> irrigation) || irrigation <= 0)
        throw invalid_argument("Frequency must be positive.");

    CerealCrop cereal(cerealName, cerealArea, cerealSeason,
                      cerealYield, fertilizer);

    VegetableCrop vegetable(vegetableName, vegetableArea,
                            vegetableSeason, vegetableYield,
                            irrigation);

    // Runtime polymorphism
    Crop* crops[2] = {&cereal, &vegetable};

    cout << "\n===== CROP DETAILS =====\n";
    for (int i = 0; i < 2; i++) {
        crops[i]->display();
    }

    cout << "\n===== FARMING ADVICE =====\n";
    Advisory* advice[2] = {&cereal, &vegetable};
    for (int i = 0; i < 2; i++) {
        advice[i]->giveAdvice();
    }

    cout << "\n===== YIELD COMPARISON =====\n";
    if (cereal > vegetable)
        cout << cerealName << " has the higher total yield.\n";
    else if (vegetable > cereal)
        cout << vegetableName << " has the higher total yield.\n";
    else
        cout << "Both crops have equal total yields.\n";

    cout << "\nTotal crops managed: "
         << Crop::getCropCount() << "\n";

    // File handling: save crop information
    ofstream outFile("farm_data.txt");
    if (!outFile)
        throw runtime_error("Unable to open file for writing.");

    outFile << "SMART FARM MANAGEMENT SYSTEM\n";
    cereal.save(outFile);
    vegetable.save(outFile);
    outFile.close();

    cout << "\nCrop data saved successfully.\n";

    // File handling: retrieve crop information
    ifstream inFile("farm_data.txt");
    if (!inFile)
        throw runtime_error("Unable to open file for reading.");

    string line;
    cout << "\n===== SAVED FARM DATA =====\n";
    while (getline(inFile, line)) {
        cout << line << '\n';
    }
    inFile.close();

} catch (const exception& e) {
    cerr << "\nError: " << e.what() << '\n';
    return 1;
}

return 0;
}