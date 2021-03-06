# TextDigester

*** LATEST VERSION: 0.1 - available from: 11/4/2017 ***

*** Winner project of the <a href="http://www.red.es/redes/es/magazin-red/eventos/i-hackathon-tecnolog%C3%ADas-del-lenguaje" target="_blank">I Hackathon de Tecnologías del Lenguaje (1st NLP Hackathon)</a>***

TextDigester is a self-contained Java library that implements several text summarization approaches relying on (and thus importing) the following libraries:

* Freeling (v 4.0): http://nlp.cs.upc.edu/freeling/  
* GATE (v 8.3): https://gate.ac.uk/  
* Deeplearning4j (v 0.7.2): https://deeplearning4j.org/  

TextDigester is developed as an Open Source Java library characterized by a modular and extensible structure. Part of the Summarization approaches implemented rely on the SUMMA document summarization library (SUMMA library http://www.taln.upf.edu/pages/summa.upf/).


## How to import TextDigester in your Java project

TextDigester is structured as a Maven project working with Java 1.8.  

Before using TextDigester in your Java application you need to:

* Install Freeling (v 4.0, http://nlp.cs.upc.edu/freeling/) together with the Freeling Java wrapper (instruction here: https://github.com/TALP-UPC/FreeLing/tree/master/APIs/java) in order to be able to parse textual contents in Java by relying on Freeling. In order to use Freeling, you should add to the <a href="https://examples.javacodegeeks.com/java-basics/java-library-path-what-is-it-and-how-to-use/" target="_blank">java.library.path</a> the local path to freeling_javaAPI.so. Remember that the Freeling 4.0 Jar dependency is imported in TextDigester by means of the POM:
```java
<!-- Freeling -->
<dependency>
	<groupId>edu.upc</groupId>
	<artifactId>freeling</artifactId>
	<version>4.0</version>
</dependency> 
```  
* Install GATE (v 8.3, https://gate.ac.uk/) and locate the GATE home and plugins directories


In order to import TextDigester in your Java program:

* Clone and compile TextDigester Maven project  
* Download the resource folder and the property file of TextDigester from http://www.backingdata.org/textdigester/TEXTDIGESTER_RESOURCES_v_0_1.tar.gz  
* Modify the property file of TextDigester by specifying the local path to the resource folder of TextDigester (downloaded in the previous tar.gz file) and the local path of your GATE installation (GAE home and GATE plugin directories)

Before using TextDigester library methods, remember to set the path to the property file of TextDigester by means of the following code:   
```java
edu.upf.taln.textdigester.setting.PropertyManager.setPropertyFilePath("/local/path/to/TextDigesterConfig.properties");
```  


## Using TextDigester to summarize documents

The following Java file is an example code of how TextDigester can be used to create the summary of a text:  
```java
package edu.upf.taln.textdigester;

import java.net.URL;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import edu.upf.taln.textdigester.importer.HTMLimporter;
import edu.upf.taln.textdigester.model.TDDocument;
import edu.upf.taln.textdigester.resource.freeling.FlProcessor;
import edu.upf.taln.textdigester.setting.LangENUM;
import edu.upf.taln.textdigester.setting.PropertyManager;
import edu.upf.taln.textdigester.setting.exception.TextDigesterException;
import edu.upf.taln.textdigester.summarizer.ConfigurableSummarizer;
import edu.upf.taln.textdigester.summarizer.SummarizationMethodENUM;
import edu.upf.taln.textdigester.summarizer.util.SummaryUtil;
import gate.Annotation;

/**
 * This class shows a typical usage pattern of TextDigester
 * 
 * @author Francesco Ronzano
 *
 */
public class CoreExample {

	private static final Logger logger = LoggerFactory.getLogger(CoreExample.class);

	public static void main(String[] args) {
		
		/* Load property file */
		PropertyManager.setPropertyFilePath("/local/path/to/TextDigesterConfig.properties");

		/* Extract main-text from HTML page and parse it */
		TDDocument HTMLdoc = null;
		try {
			HTMLdoc = HTMLimporter.extractText(new URL("http://www.ara.cat/economia/clients-seran-proper-negoci-Telefonica_0_1749425231.html"));
			logger.debug("TEXT: " + HTMLdoc.getOriginalText());

		} catch (Exception e) {
			e.printStackTrace();
		}

		/* Process text document by identifying its language and then parsing its contents by Freeling */
		LangENUM languageOfHTMLdoc = FlProcessor.getLanguage(HTMLdoc.getOriginalText());

		HTMLdoc = FlProcessor.parseDocumentGTSentences(HTMLdoc, languageOfHTMLdoc);

		/* Try different summarization methods; each one of them returns a map with as key a sentence gate.Annotation instance and 
		 * as value the relevance score assigned to that sentence (a sentence with an higher relevance score is more suitable to 
		 * be included in an extractive summary of the initial text).
		 * The documents to summarize and the related textual annotations are represented by means of the GATE textual annotation 
		 * data model - https://gate.ac.uk/sale/tao/splitch5.html - https://gate.ac.uk/releases/latest/doc/javadoc/. 
		 * List of summarization methods available - in the enumeration: edu.upf.taln.textdigester.summarizer.SummarizationMethodENUM */
		
		try {
			/* By means of the ConfigurableSummarizer.summarize static method it is possible to invoke the different summarization methods 
			 * implemented by TextDigester. 
			 */
			
			// Summarization method: Centroid_TFIDF - represent sentences by means of their TF-IDF vectors. Compute the centroid of all sentence 
			// TF-IDF vectors and rank sentences with respect to their cosine similarity to the centorid vector.
			Map<Annotation, Double> orderedSentences_Centroid_TFIDF = ConfigurableSummarizer.summarize(HTMLdoc, languageOfHTMLdoc, SummarizationMethodENUM.Centroid_TFIDF);

			// Summarization method: Centroid_EMBED represent sentences by means of their EMBEDDING vectors (computed by means of Doc2Vec implementation of Deeplearning4j).
			// Compute the centroid of all sentence EMBEDDING vectors and rank sentences with respect to their cosine similarity to the centorid vector.
			Map<Annotation, Double> orderedSentences_Centroid_EMBED = ConfigurableSummarizer.summarize(HTMLdoc, languageOfHTMLdoc, SummarizationMethodENUM.Centroid_EMBED);

			// Summarization method: TextRank_TFIDF - Execute the TextRank algorithm (https://web.eecs.umich.edu/~mihalcea/papers/mihalcea.emnlp04.pdf) over the sentences
			// by computing the similarity among sentences relying on the cosine similarity of the respective TF-IDF vectors.
			Map<Annotation, Double> orderedSentences_TextRank_TFIDF = ConfigurableSummarizer.summarize(HTMLdoc, languageOfHTMLdoc, SummarizationMethodENUM.LexRank_TFIDF);

			// Summarization method: TextRank_EMBED - Execute the TextRank algorithm (https://web.eecs.umich.edu/~mihalcea/papers/mihalcea.emnlp04.pdf) over the sentences
			// by computing the similarity among sentences relying on the cosine similarity of the respective EMBEDDING vectors (computed by means of Doc2Vec implementation 
			// of Deeplearning4j).
			Map<Annotation, Double> orderedSentences_TextRank_EMBED = ConfigurableSummarizer.summarize(HTMLdoc, languageOfHTMLdoc, SummarizationMethodENUM.LexRank_EMBED);

			// Summarization method: FirstSim - Rank the sentences with respect to their similarity to the first sentence of the document by computing the similarity 
			// among sentences relying on the cosine similarity of the respective TF-IDF vec
			Map<Annotation, Double> orderedSentences_FirstSim = ConfigurableSummarizer.summarize(HTMLdoc, languageOfHTMLdoc, SummarizationMethodENUM.FirstSim);

			// Summarization method: TFscore - Rank sentences with respect to the sum of their TF scores
			Map<Annotation, Double> orderedSentences_TFscore = ConfigurableSummarizer.summarize(HTMLdoc, languageOfHTMLdoc, SummarizationMethodENUM.TFscore);

			// Summarization method: Centroid_TFIDF_SUMMA - Rank sentences with respect to the sum of their TF scores
			Map<Annotation, Double> orderedSentences_Centroid_TFIDF_SUMMA = ConfigurableSummarizer.summarize(HTMLdoc, languageOfHTMLdoc, SummarizationMethodENUM.Centroid_TFIDF_SUMMA);

			// Summarization method: Position - Rank sentences with respect to their position in the document to summarize
			Map<Annotation, Double> orderedSentences_Position = ConfigurableSummarizer.summarize(HTMLdoc, languageOfHTMLdoc, SummarizationMethodENUM.Position);

			// Summarization method: SemScore - Rank sentences with respect to their semantic score
			Map<Annotation, Double> orderedSentences_SemScore = ConfigurableSummarizer.summarize(HTMLdoc, languageOfHTMLdoc, SummarizationMethodENUM.SemScore);

			// Print the text of one of these summaries
			Map<Annotation, Double> orderedSentences_SemScore_top20perc = SummaryUtil.getSummary(orderedSentences_SemScore, HTMLdoc, 20d);
			System.out.println("SUMMARY: \n " + SummaryUtil.getStringSummaryText(orderedSentences_SemScore_top20perc, HTMLdoc));


		} catch (TextDigesterException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
		
		/* Load property file */
		PropertyManager.setPropertyFilePath("/local/path/to/TextDigesterConfig.properties");

		/* Extract main-text from HTML page and parse it */
		TDDocument HTMLdoc_1 = null;
		try {
			HTMLdoc_1 = HTMLimporter.extractText(new URL("http://www.ara.cat/cultura/llista-tots-nominats-als-Oscars_0_1750025056.html"));
			logger.debug("TEXT: " + HTMLdoc.getOriginalText());

		} catch (Exception e) {
			e.printStackTrace();
		}

		TDDocument HTMLdoc_2 = null;
		try {
			HTMLdoc_2 = HTMLimporter.extractText(new URL("http://www.ara.cat/cultura/moonlight-guanya-oscars_0_1750025053.html"));
			logger.debug("TEXT: " + HTMLdoc.getOriginalText());

		} catch (Exception e) {
			e.printStackTrace();
		}

		/* Process text document by identifying its language and then parsing its contents by Freeling */
		LangENUM languageOfHTMLdoc_1 = FlProcessor.getLanguage(HTMLdoc_1.getOriginalText());
		LangENUM languageOfHTMLdoc_2 = FlProcessor.getLanguage(HTMLdoc_2.getOriginalText());

		HTMLdoc_1 = FlProcessor.parseDocumentGTSentences(HTMLdoc_1, languageOfHTMLdoc_1);
		HTMLdoc_2 = FlProcessor.parseDocumentGTSentences(HTMLdoc_2, languageOfHTMLdoc_2);


		/* Try different summarization methods that return a map with key a sentence Annotation instance and value the relevance score assigned to that sentence
		 * List of summarization methods available - in the class: edu.upf.taln.textdigester.summarizer.SummarizationMethodENUM */

		List<TDDocument> docList = new ArrayList<TDDocument>();
		docList.add(HTMLdoc_1);
		docList.add(HTMLdoc_2);

		try {

			// Summarization method: CentroidMultiDoc_TFIDF
			Map<Entry<Annotation, TDDocument>, Double> orderedSentences_CentroidMultiDoc_TFIDF = ConfigurableSummarizer.summarizeMultiDoc(docList, languageOfHTMLdoc, SummarizationMethodENUM.CentroidMultiDoc_TFIDF);

			// Summarization method: CentoridMultiDoc_EMDBED
			Map<Entry<Annotation, TDDocument>, Double> orderedSentences_CentoridMultiDoc_EMDBED = ConfigurableSummarizer.summarizeMultiDoc(docList, languageOfHTMLdoc, SummarizationMethodENUM.CentoridMultiDoc_EMDBED);


			// Print the text of one of these summaries
			Map<Entry<Annotation, TDDocument>, Double> orderedSentences_SemScore_top20perc = SummaryUtil.getSummary(orderedSentences_CentoridMultiDoc_EMDBED, docList, 20d);
			System.out.println("SUMMARY: \n " + SummaryUtil.getStringSummaryText(orderedSentences_SemScore_top20perc));


		} catch (TextDigesterException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

	}

}
```  




