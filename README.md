# Github Mining
Github mining scripts used for the following paper during the 19th International Conference on Mining Software Repositories (MSR '22):
> Mining the Usage of Reactive Programming APIs: A Mining Study on GitHub and Stack Overflow.

Complementary scripts, also utilized during the paper production, are available in:
* [Stack Overflow Mining](https://github.com/carloszimm/so-mining-msr22)
* [Operators Scraping](https://github.com/carloszimm/rx-scraping-msr22)

## Data
Under the folders in `/assets`, data either genereated by or collected for the scripts execution can be found. The table gives a brief description of each folder:

| Folder   | Description         |
| :------------- |:-------------|
| false-positives | Data related to the checking of false positives through Java collection-like libraries |
| operators-search | Includes the results for the Rx libraries' operator search |
| operators | Includes JSON files consisting of Rx libraries' operators |
| repo-retrieval | Contains data about the GitHub repositories retrieved and processed |
| result-search | Contains information about each dependent repository with >= 10 stars|
| result-summary | Contains a summary of all rx distribution, including their total of dependent repositories, those with 0 stars and those with >=10 stars  |
| so-data | Stack Overflow data colleted through the Stack Mining scripts |

The file `Programming_Languages_Extensions.json` contains a list of extensions used by several languages. In the paper, the following entries were utilized:
* Java (for RxJava)
* JSX, JavaScript, and TypeScript (for RxJS)
* Swift (for RxSwift)

As detailed in the paper, the `repo-retrieval` result does not provide the actual GitHub repositories given the size constraints to upload them here (even if they are compressed as tarball files). Instead, under each rx library folder inside `repo-retrieval` (e.g., `/assets/repo-retrieval/rxjava`), there is a file called `list_of_files.json` containing an array of objects with the following info that can be used to download the exactly same files (that must me place in a subfolder `/archives` relative to `list_of_files.json`):
| Entry   | Description         |
| :------------- |:-------------|
| owner | the owner of the repository |
| repoName | the repository name |
| repoFullName | the full repository name (i.e., with the _owner\_name/_ as a prefix) |
| branch | the default branch |
| fileName | the name of the tarball file |
| fileSize | the files' size in bytes |
| url | the url to download the tarball file with the SHA1 of the last commit already set |

## Execution
### Requirements
Most of the scripts utilize Golang (mainly) and Nodejs and they have be executed the following versions:
* Go v1.17.5
* Node.js v14.17.5

### Scripts
The Go scripts are available under the `/cmd` folder. All of them are structured to be executed at the root of the repository.
Before execution of any Go script, one must run the following command in a terminal to install all the dependencies:
```sh
go mod tidy
```

**operator-search**

Script to search for the Rx operators.
```sh
go run cmd/operator-search/main.go
```
&ensp; :floppy_disk: After execution, the result is available at `assets/operators-search`.

> **Note**: This script also accepts an additional command flag (**-checkfalsepositives**) which changes the behavior of the search to also inspect files looking for Java collection-like libraries ([Java Streams](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html), [Eclipse Collections](https://github.com/eclipse/eclipse-collections), [Apache's CollectionUtils](https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/CollectionUtils.html), and [Guava's Collections2](https://guava.dev/releases/23.0/api/docs/com/google/common/collect/Collections2.html)). As explained in the paper, the regex method doesn't guarantee that false positives aren't introduced in the mining process; however, given that Rx can wrap any type of value, we checked Java files, the one with more inspected projects, to make sure that few false positives were being counted. The script prints the files that have both RxJava import and the collection-like libraries at the same time, and the result is already saved at `collection-like_files.txt` file under `assets/false-positives`. In total, 156 files were found and, from those, 16 (10%) were manually verified to check false positives (results are available at the paper's GitHub Mining Section). The list of the 16 sample is available in `assets/false-positives/collection-like_sample.txt` which was generated with the help of [RANDOM.ORG](https://www.random.org/). Moreover, a copy of those files is also available at `assets/false-positives/sample-files/`. To help the manual process checking, the script additionally reads this list of 16 files and stores the operators' frequencies (>0), true and false positives, in `assets/false-positives/collection-like_count.txt`. Before executing the script with the flag, the RxJava library must be set in the [configuration](#configuration).

**repo-retrieval**

Script to retrieve the repositories to be mined.
```sh
go run cmd/repo-retrieval/main.go
```
&ensp; :floppy_disk: After execution, the result is available at `assets/repo-retrieval`.

**repo-search**

Script to search for repositories using selected rx libraries e save that information in a file, so repo-retrieval can proceed.
```sh
go run cmd/repo-search/main.go
```
&ensp; :floppy_disk: After execution, the result is available at `assets/repo-search`.

**repo-summary**

Script to create a summary of all rx distribution, including their total of dependent repositories, those with 0 stars and those with >=10 stars.
```sh
go run cmd/repo-summary/main.go
```
&ensp; :floppy_disk: After execution, the result is available at `assets/repo-search`.

#### Configuration
The majority of the Go scripts depend on entries in a JSON object located in `/configs/config.json`. This object has the following structure(this is the object present by default in config.json):
```yaml
{
    "tokens": [],
    "distribution": "RxJS",
    "min_stars": 10,
    "increase_factor": 50,
    "file_extensions": ["JSX", "JavaScript", "TypeScript"]
}
```
Where:
* **tokens(array of strings)**: GitHub tokens used mainly in scripts involving GitHub queries. Those tokens are exploited to create workers, so the queries can be executed more quickly. During the paper's executions, we leveraged three GitHub tokens/workers;
* **distribution(string)**: the distribution/library (RxJava, RxJS, and RxSwift) to be considered in the current execution of some scripts;
* **min_stars(integer)**: the minimum number of stars to be used in the search for Rx-dependent repositories;
* **increase_factor(integer)**: used to control the factor by which the star intervals are contructed until reaching the limit (found by issuing a previous query where the number of stars is descendingly sorted). It is also used in the search for Rx-dependent repositories;
*  **file_extensions(array of strings)**: lists the entries of `Programming_Languages_Extensions.json` file that should be considered in repo-retrieval script. The [Data](#data) section describes the entries leveraged in the paper.

#### Nodejs scripts

The Nodejs scripts, in turn, are available under the `/scripts/charts` folder. They were utilized post mining to generate charts and
data (CSV). Their results are available at `/scripts/charts/results`.
Before execution of any Node script, one must run the following command in a terminal to install all the dependencies:
```sh
npm install
```
All the Node scripts should use `/scripts/charts` as the working directory.

##### generate-similarity
Script to generate charts and data related to RQ3.
```sh
node generate-similarity
```
&ensp; :floppy_disk: By the end of execution, the results are available in `/scripts/charts/results/similarity`.
The script produces many outputs and they can be generalized as:
<br/>&emsp;&emsp;&emsp; :white_medium_small_square: **frequencies\__[relevant topic]_\__[rx library]_.csv**: operators frequencies of the most relevant topics in the rx libraries analyzed. Usage frequencies acquired from Stack Overflow posts;
<br/>&emsp;&emsp;&emsp; :white_medium_small_square: **similarities\__[relevant topic]_\__[rx library]_.csv**: contains the operators(and their frequencies) of the most relevant topics according to the rx libraries analyzed in which their frequency placement matches the frequency placement of the rx operators' frequencies collected in GitHub projects;
<br/>&emsp;&emsp;&emsp; :white_medium_small_square: **similarities\__[ 'leastUsed' | 'mostUsed' ]_\__[relevant topic]_\__[rx library]_.csv**: close to the results above but considering the top least and most used operators and disconsidering their placement(order);
<br/>&emsp;&emsp;&emsp; :white_medium_small_square: **similarity.png**: percentage of similarity when comparing the operators (sorted by their frequency) of the most relevant topics and the operators (also sorted by their frequencies) found in GitHub repositories. This figure takes into account the order of appearance of each operator (their placement) according to their frequency;
<br/>&emsp;&emsp;&emsp; :white_medium_small_square: **similarity\__[ 'leastUsed' | 'mostUsed' ]_.png**: percentage of similarity when comparing the top least and most used (according to their frequencies) operators of the most relevant topics and the top least and most used operators (sorted by their frequencies) found in GitHub repositories. This figure does not take into account the order of appearance of each operator (their placement) according to their frequency. It only considers if there is a match between the lists of least or most used operators;

Where, _[relevant topic]_ = {Dependency Management, Introductory Questions, iOS Development} and _[rx library]_ = {RxJava, RxJS, RxSwift}

---
##### generate_frequencies
Script to generate charts and data related to the frequencies presented in RQ1.
```sh
node generate_frequencies
```
&ensp; :floppy_disk: By the end of execution, the results are available in `/scripts/charts/results/frequency`.
The script produces many outputs and they can be summarized as:
<br/>&emsp;&emsp;&emsp; :white_medium_small_square:**frequency\__[rx library]_\__[ 'topLeastUsed' | 'topMostUsed' ]_.png**: charts showing the top least and most used operator, according to their frequency, of each Rx library analyzed;
<br/>&emsp;&emsp;&emsp; :white_medium_small_square:**frequency\__[rx library]_.[ csv | json ]** - CSV and JSON files containing operator frequencies of each Rx library analyzed.

Where, _[rx library]_ = {RxJava, RxJS, RxSwift}

---
##### generate_utilization
Script to generate a chart showing the percentage of utilization when combine all frequencies from operators of the three studied libraries (not present in the paper only the percentage).
```sh
node generate_utilization
```
&ensp; :floppy_disk: By the end of execution, the result is placed in `/scripts/charts/results/utilization`

---
##### generate_utilization_distribution
Script to generate a chart showing the percentage of utilization of the operators in each Rx library: RxJava, RxJS, and RxSwift.
```sh
node generate_utilization_distribution
```
&ensp; :floppy_disk: By the end of execution, the result is placed in `/scripts/charts/results/utilization_perDistribution`
