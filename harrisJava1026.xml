import java.awt.Color;
import java.io.File;
import java.io.FileOutputStream;
import java.io.OutputStreamWriter;
import java.io.PrintWriter;
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.*;

//import org.junit.Assert;

import lotus.domino.Database;
import lotus.domino.EmbeddedObject;
import lotus.domino.Item;
import lotus.domino.RichTextItem;
import lotus.domino.View;
//import lotus.domino.ViewEntry;
import lotus.domino.ViewEntryCollection;

import org.jdom.Document;
import org.jdom.Element;
import org.jdom.DocType;


import org.apache.pdfbox.util.PDFMergerUtility;
import org.apache.pdfbox.cos.COSName;
import org.apache.pdfbox.pdmodel.PDDocument;
import org.apache.pdfbox.pdmodel.PDDocumentCatalog;
import org.apache.pdfbox.pdmodel.PDPage;
import org.apache.pdfbox.pdmodel.PDResources;
import org.apache.pdfbox.pdmodel.graphics.color.PDGamma;
import org.apache.pdfbox.pdmodel.graphics.optionalcontent.PDOptionalContentGroup;
import org.apache.pdfbox.pdmodel.graphics.optionalcontent.PDOptionalContentProperties;
import org.apache.pdfbox.pdmodel.interactive.action.type.PDActionURI;
import org.apache.pdfbox.pdmodel.interactive.annotation.PDAnnotationLine;
import org.apache.pdfbox.pdmodel.interactive.annotation.PDAnnotationLink;
import org.apache.pdfbox.pdmodel.interactive.annotation.PDAnnotationSquareCircle;
import org.apache.pdfbox.pdmodel.interactive.annotation.PDAnnotationTextMarkup;
import org.apache.pdfbox.pdmodel.interactive.annotation.PDBorderStyleDictionary;
import org.apache.pdfbox.pdmodel.markedcontent.PDPropertyList;

import org.apache.pdfbox.pdmodel.common.PDRectangle;
import org.apache.pdfbox.pdmodel.edit.PDPageContentStream;
import org.apache.pdfbox.pdmodel.font.PDType1Font;
import org.apache.pdfbox.pdmodel.font.PDFont;

import java.util.Date;
import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.Calendar;

public class CreateXMLFileJ {
	public static Utilities util;
	public static lotus.domino.Database dbThis;
	public static lotus.domino.ViewEntryCollection vwEntries;
	public static lotus.domino.ViewEntry veBack;
	public static lotus.domino.ViewEntry veNext;
	public static lotus.domino.DocumentCollection dcProcessed;
	public static lotus.domino.Document docBack;
	public static ConfigParameters config;
	
	private static lotus.domino.Log aLog;
	private static String topKey;
	private static String tmpBatch;
	private static Vector tmpName;
	static boolean results;
	private static View vwFolder;
	private static String exportFolder;
	private static String msg;
	
	private static Integer loopTreatmentPlan;
	private static Element rootElem;
	
	public static String validDiagnosisCode;  
	public static long docCount = 0;
	public static int docCountROW = 0;
	public static boolean isErr = false;
	public static int serviceDoc = 0;
	public static int serviceItem = 0;
	public static String getServiceItem = "";
	public static int serviceItemTotal = 0;
	public static String tmpVal;
	public static String tmpVal2;
	public static String serviceRow; 
	
	public static boolean initXML(lotus.domino.Session session, lotus.domino.AgentContext agentContext, lotus.domino.Database dbPass, String strKey ) {
		
		boolean result = false;
		dbThis = dbPass;
		
		// - Setup our configParameter and util objects
		try {
			// - Create the config object -
			config = new ConfigParameters( session, agentContext, dbThis );
			aLog = config.getLog();
			// - Create the Utilities -
			util = new Utilities( session, agentContext, config, strKey );
			System.out.println("InitXML Done. Returning ");
			result = true;
			
		} catch (Exception e1) {
			//error handling here
			if ( util.logLevel > 0 ) {
				msg = "General error encountered in createXMLFile, part 1: " + e1.getMessage();
				try {
					aLog.logError(e1.hashCode(), msg);
				}catch(Exception ex){
					System.out.println(msg);
					e1.printStackTrace();
				}
			}
			sendErrorNotice( e1, dbThis, "Module: initXML." );
			isErr = true;
		}
		return result;
	}

	
	public static long createXMLFile(lotus.domino.Session session, lotus.domino.AgentContext agentContext, lotus.domino.Database dbPass, String strKey, boolean validateXML ) {
		
		boolean displayXML = false ;			//Sets whether to save the xml or generate it as output to the console. T=Display, F=Save
		boolean isSaved = false;
		boolean isValidXML = true;
		int counter ;
		counter = 1; 
		validDiagnosisCode = "";
		
		topKey = strKey;
		System.out.println("Starting Create XMLFile: ");
		// -Get a handle on the batch number for this file-
		int batchNum;
		try {
			
			if ( util.logLevel > 1 ) {
				msg ="getting config docs";
				try {
					aLog.logAction(msg);
				}catch(Exception ex){
					System.out.println(msg);
				}
			}
			lotus.domino.Document docConfig;
		docConfig = config.getConfigDoc(strKey.toUpperCase() + "-BATCHCOUNTER");
			if ( docConfig == null ) {
				if ( util.logLevel > 1 ) {
					msg = "Creating new batch number and incrementing configuration document.";
					try {
						aLog.logAction(msg);
					} catch(Exception ex) {
						System.out.println(msg);
					}
				}
				docConfig = dbThis.createDocument();
				docConfig.replaceItemValue("Form", "profile");
				docConfig.replaceItemValue("keyword", strKey.toUpperCase() + "-BATCHCOUNTER");
				docConfig.replaceItemValue("description", util.masterDate);
				tmpBatch = "0";
			}
			String lastMasterDate = util.getValue( docConfig, "description" );
			if ( lastMasterDate.equals( util.masterDate ) )
			{
				// - Same date, new batch -
				if ( util.logLevel > 2 ) {
					msg = "Batch number configuration document master date = this master date.";
					try {
						aLog.logAction(msg);
					} catch(Exception ex) {
						System.out.println(msg);
					}
				}
				tmpBatch = util.getValue( docConfig, "values" );
			} else {
				// - New date, restart batch counter -
				if ( util.logLevel > 2 ) {
					msg = "Batch number configuration document master date ( " + util.getValue( docConfig, "description" ) + " ) != this master date ( " + util.masterDate + " ).";
					try {
						aLog.logAction(msg);
					} catch(Exception ex) {
						System.out.println(msg);
					}
				}
				docConfig.replaceItemValue("description", util.masterDate );
				tmpBatch = "0";
			}
			try
			{
				batchNum = Integer.parseInt(tmpBatch);
				batchNum++;
				if ( util.logLevel > 0 ) {
					msg = "New batch number: " + batchNum;
					try {
						aLog.logAction(msg);
					} catch(Exception ex) {
						System.out.println(msg);
					}
				}
			}
			catch ( NumberFormatException ex )
			{
				if ( util.logLevel > 0 ) {
					msg = "Error occurred attempting to increment the batch number. Defaulted to 1";
					try {
						aLog.logAction(msg);
					} catch(Exception el) {
						System.out.println(msg);
					}
				}
				batchNum = 1;
			}
			docConfig.replaceItemValue("description", "" + util.masterDate);
			docConfig.replaceItemValue("values", "" + batchNum);
			docConfig.save(true,true,true);
			if ( util.logLevel > 1 ) {
				msg = "Formatting batch number. " + batchNum;
				try {
					aLog.logAction(msg);
				} catch(Exception el) {
					System.out.println(msg);
				}
			}

			util.setBatch( batchNum, util.logLevel );
		} catch (Exception e2) {
			//error handling here
			if ( util.logLevel > 0 ) {
				msg ="General error encountered in createXMLFile, part 2:";
				try {
					aLog.logError(e2.hashCode(), msg);
				} catch(Exception el) {
					System.out.println(msg);
				}
			}
			e2.printStackTrace();
			isErr = true;
			sendErrorNotice( e2, dbThis, "Module: createXMLFile, part 2." );
		}
		
		if (isErr)
			return 0;
		
		//get root element name
		String luKey = strKey + "-ROOT-ELEMENT";
		Vector vecTmp = config.getKeyList(luKey);
		String rootName = (String)vecTmp.elementAt(0);
		
		if ( util.logLevel > 0 ) {
			msg = "Root element: " + tmpName;
			try {
				aLog.logAction(msg);
			} catch(Exception el) {
				System.out.println(msg);
			}
		}

		rootElem = new Element(rootName);

		luKey = strKey + "-" + tmpName;
		String tmpName = (String)vecTmp.elementAt(0);
		System.out.println("Root start - is it root or next level?  " + tmpName);
		
		// - Find folders -
		try {
			luKey = topKey.toUpperCase();
			luKey += "-FOLDER";
			if ( util.logLevel > 1 ) {
				msg ="Get folder names from config: " + luKey;
				try {
					aLog.logAction(msg);
				} catch(Exception el) {
					System.out.println(msg);
				}
			}
			vecTmp = config.getParameterList(luKey);
			if ( util.logLevel > 0 ) {
				msg ="folder names: " + vecTmp.toString();
				try {
					aLog.logAction(msg);
				} catch(Exception el) {
					System.out.println(msg);
				}
			}
			exportFolder = (String)vecTmp.elementAt(0);
			if ( util.logLevel > 1 ) {
				msg ="exportFolder: " + exportFolder;
				try {
					aLog.logAction(msg);
				} catch(Exception el) {
					System.out.println(msg);
				}
			}
			
			vwFolder = dbThis.getView(exportFolder);
			if (vwFolder == null) {
				if ( util.logLevel > 0 ) {
					msg ="Unable to locate export folder : " + exportFolder + " in dbThis: " + dbThis.toString();
					try {
						aLog.logAction(msg);
					} catch(Exception el) {
						System.out.println(msg);
					}
				}
				sendErrorNotice( new Exception("Unable to locate export folder : " + exportFolder + " in dbThis: " + dbThis.toString()), dbThis, "Module: createXMLFile, part 2." );
				return 0;
			}
			if ( util.logLevel > 1 ) {
				msg ="vwFolder: " + vwFolder.getName();
				try {
					aLog.logAction(msg);
				} catch(Exception el) {
					System.out.println(msg);
				}
			}
			vwEntries = vwFolder.getAllEntries();
			
			// - Create an empty document collection to contain successfully exported documents -
			vecTmp = new Vector();
			vecTmp.add("«»");
			dcProcessed = vwFolder.getAllDocumentsByKey(vecTmp, true);
			msg ="vwFolder Count: " + dcProcessed.getCount();
			
			System.out.println(msg);
			
			
		} catch (Exception ne) {
			//error handling here
			sendErrorNotice( ne, dbThis, "Module: createXMLFile, find folders." );
			if ( util.logLevel > 0 ) {
				msg ="General error encountered in createXMLFile, find folders:";
				try {
					aLog.logError(ne.hashCode(), msg);
				} catch(Exception el) {
					System.out.println(msg);
				}
			}
			ne.printStackTrace();
			isErr = true;
		}
		
		
		if (isErr)
			return 0;

		// - Create an XML object starting with the root element -
		if ( util.logLevel > 1 ) {
			msg = "Creating the XML Document.";
			try {
				aLog.logAction(msg);
			} catch(Exception el) {
				System.out.println(msg);
			}
		}
		
		DocType daDocType = util.getDocType( util.dtdURL );

		// - this call will append any child elements to the passed root element -
		if ( util.logLevel > 1 ) {
			msg = "CreateXMLFile method, passing params to createXMLFile - tmpName: " + tmpName;
			try {
				aLog.logAction(msg);
			} catch(Exception el) {
				System.out.println(msg);
			}
		}
		
		System.out.println("ceateelements: " + rootName);
		rootElem = createElementsFromConfig(rootName);
		//rootElem = createElementsFromConfig("row");
		if (isErr)
			return 0;
		if ( util.logLevel > 1 ) {
			msg ="Complete XML: " + rootElem.toString();
			try {
				aLog.logAction(msg);
			} catch(Exception el) {
				System.out.println(msg);
			}
		}

		// - create the XML document using the DOM object created above -
		Document theXMLDoc = new Document( rootElem, daDocType );
		
		if (isErr)
			return 0;
		if (util.logLevel > 0 ) {
			msg ="Processed " + docCount + " documents into:\r" + rootElem.toString();
			try {
				aLog.logAction(msg);
			} catch(Exception el) {
				System.out.println(msg);
			}
		}
		
		if (docCount< 1 ) {
			// - No documents were processed, we are finished -
			if ( util.logLevel > 0 ) {
				msg = "No documents were processed using key: " + strKey;
				try {
					aLog.logAction(msg);
				} catch(Exception el) {
					System.out.println(msg);
				}
			}
			return docCount;
		}
		
		if ( util.logLevel > 0 ) {
			msg ="Output the XML";
			try {
				aLog.logAction(msg);
			} catch(Exception el) {
				System.out.println(msg);
			}
		}
		if (theXMLDoc != null) {		/* Save the XML or print it to console - set boolean displayXML to display or save*/
			isSaved = XML_SaveToFile.saveXML( util, theXMLDoc, displayXML, docCount, util.logLevel);
			if (isErr) {
				backOutBatch(util, dbThis);
				return 0;
			}
		}
		else
		{
			sendErrorNotice( new Exception("theXML is null"), dbThis, "Module: createXMLFile." );
			if ( util.logLevel > 0 ) {
				msg ="theXML is null";
				try {
					aLog.logError((int)1585, msg);
				} catch(Exception el) {
					System.out.println(msg);
				}
			}
			backOutBatch( util, dbThis );
		}

		
		return docCount;
	}
	
	
	public static Element createElementsFromConfig(String strKey) {
		/*
		 * This method uses a passed key to locate config documents and apply those in building XML elements
		 * Iterates through all key values found creating child elements by calling buildElementFromConfig
		 * Will locate the relevant folder containing documents to be exported
		 */
		
		// - Find the key-value list used to create elements at this level. -
		if ( util.logLevel > 1 )
			System.out.println ("Running: createElementsFromConfig('" + strKey + "')");
		String luKey = topKey.toUpperCase();
		luKey += "-ROOT-ELEMENT";
		Vector vecTmp = config.getKeyList(luKey);
		if ( vecTmp == null )
			return null;
		
			
		String elemName = (String)vecTmp.elementAt(0);
		System.out.println ("string elemName right before root" + elemName);
		
		//String elemName1 = (String)vecTmp.elementAt(1);

		rootElem = new Element(elemName);
		if ( util.logLevel > 0 )
			System.out.println ("rootElem: " + rootElem.getName());
		luKey = topKey.toUpperCase();
		luKey += "-" + elemName;
		vecTmp = config.getKeyList(luKey);
		if ( vecTmp == null )
			return null;
		elemName = (String)vecTmp.elementAt(0);
		
//		String thirdTier = "IHFS-EXPORT"
		Vector secondTier = config.getKeyList(luKey);
		//vecTmp = config.getKeyList("XML-IHFS-EXPORT-ADULT-ROOT-ROW");
		
		//System.out.println("DC second tier " + thirdTier );
		
		//elemName1 = (String)vecTmp.elementAt(1);
		
		//Element ROW = new Element("row");				
		//Element ROW;
		
		//rootElem.addContent(nextR);
		//nextS.addContent(tmpElem);
		//rootElem.addContent(nextS);

		//Element CollectRows = new Element("space");
		
			try {
			// - Loop thru all documents creating an Element for each as configured by the key-value list -
			veBack = vwEntries.getFirstEntry();
			
			
			
			while ( veBack != null ) {
				
				/*
				if (serviceDoc==1) {
					for  (int dc=0; dc<=1;dc++) {
						}
				}
			
				 */
				
				
				Element ROW = new Element( "row");
				docCountROW++;
				/*
				if (docCountDOS >1) {
					System.out.println ("loop loop: " + docCountDOS);
					
					rootElem.addContent(ROW);
				}
				*/
				
//				rootElem.addContent(ROW);
				
				//dcloop
				/*
							for (int dc=0; dc<=1;dc++){
								if (dc==0) {
									String misc = "x";
							} else {
									elemName = "clientinfoy";
							}
				*/
				
				if ( util.logLevel > 1 )
					//1025 System.out.println ("docBack: " + docBack.toString());
				if ( util.logLevel > 2 ) {
					msg ="createElementsFromConfig calling buildElementFromConfig('" + strKey + "', '" + elemName + "')";
					try {
						aLog.logAction(msg);
					} catch(Exception el) {
						System.out.println(msg);
					}
				}
				
				
				/*
				if (elemName.equals("treatmentplan")) {
					//int tempTreatCount = 0;
					int tempTreatMax = 1;
					for (int tempTreatCount=0; tempTreatCount<tempTreatMax;tempTreatCount++){
						Element tmpElem2 = buildElementFromConfig(docBack, strKey, elemName, tempTreatCount);	
						ROW.addContent(tmpElem2);	
					}
				} 
			*/
 
				
				/*
				//Element tmpElem;
				if (elemName.contains("treatmentplan")) {
					for (int tpx=0; tpx<1; tpx++) {	
						Element tmpElem = (buildElementFromConfig(docBack, strKey, elemName, 1));
					}
				} else {
					Element tmpElem = buildElementFromConfig(docBack, strKey, elemName, 1);
				}
				*/
				
				
				
				/*1029
				if (serviceDoc==1) {
					System.out.println ("servicedoc loop: "); serviceDoc=0;
				}else{
					veNext = vwEntries.getNextEntry(veBack);
					docBack = veBack.getDocument();
				}
			*/
				
				Element tmpElem = (buildElementFromConfig(docBack, strKey, elemName, 1));
				
					
				if ( tmpElem != null ) {
					if (isErr)
						return null;
					//	Element nextR = new Element("x");					
					//	rootElem.addContent(nextR);

					//1025 
					
					if(validDiagnosisCode.equals("notvalid")){
						System.out.println ("Diagnosis or treatment not valid");
						tmpElem.removeContent();
						validDiagnosisCode = "";
					} else if (serviceDoc == 1) {
						//ROW.addContent(tmpElem);
						//ROW.addContent(tmpElem);
						
					}else {
						ROW.addContent(tmpElem);
					};

					
					
					if ( util.logLevel > 1 )
						System.out.println ("Created child element: " + tmpElem.getName());
					dcProcessed.addDocument(docBack);
					docCount++;
				} else {
					sendErrorNotice( new Exception("General error encountered in createElementsFromConfig"), dbThis, "Module: createElementsFromConfig - null element returned for buildElementFromConfig('" + strKey + "', '" + elemName + "')" );
					return null;
				}
							
				
		//dc	
				
				for (int st=1; st<secondTier.size();st++){
					                   
					Element tmpElem1 = buildElementFromConfig(docBack, strKey, secondTier.elementAt(st).toString(), 2);
					//	ROW.addContent(tmpElem1);
				
					
					if(validDiagnosisCode.equals("notvalid")){
						System.out.println ("Dignosis not valid");
						tmpElem.removeContent();
						validDiagnosisCode = "";
					} else {
						ROW.addContent(tmpElem1);
					};

					
					//rootElem.addContent(tmpElem1);
				}
				
						
				
				//CollectRows.addContent(ROW);
								
				//Element ROW = Element("sdf");
				//Element ROW = "row";
				rootElem.addContent(ROW);
				veBack = veNext;
			}
		} catch (Exception e2) {
			//error handling here
			sendErrorNotice( new Exception("General error encountered in createElementsFromConfig"), dbThis, "Module: createElementsFromConfig." );
			if ( util.logLevel > 0 ) {
				msg ="General error encountered in createElementsFromConfig:";
				try {
					aLog.logError(e2.hashCode(), msg);
				} catch(Exception el) {
					System.out.println(msg);
				}
			}
			e2.printStackTrace();
			isErr = true;
		}
		if ( util.logLevel > 1 ) {
			msg ="rootElem: " + rootElem.toString();
			try {
				aLog.logAction(msg);
			} catch(Exception el) {
				System.out.println(msg);
			}
		}
//		} //dc add x25
		
		Element trailer = new Element( "trailer");
		
		
		Element rowtotal = new Element( "rowtotal");
		
		rowtotal.setText(String.valueOf(docCountROW));
		trailer.addContent(rowtotal);
		
		
		rootElem.addContent(trailer);
		
		return rootElem;
	}
	
	public static Element buildElementFromConfig(lotus.domino.Document docBack, String strKey, String elemName, int getRound )
	{
		String randomStr;
		
		//create child elements from strKey
		if ( util.logLevel > 2 ) {
			System.out.println("buildElementFromConfig: elemName is " + elemName);
		}
		if ( util.logLevel > 2 ) {
			msg ="Running: buildElementFromConfig('" + strKey + "', '" + elemName + "')";
			try {
				aLog.logAction(msg);
			} catch(Exception el) {
				System.out.println(msg);
			}
		}
		String luKey = topKey.toUpperCase();
		if ( !strKey.equals("")) {
			luKey += "-" + strKey.toUpperCase();
			luKey += elemName.equals("") ? "" : "-" + elemName.toUpperCase();
		} else {
			luKey += elemName.equals("") ? "" : "-" + elemName.toUpperCase();
		}
		if ( util.logLevel > 2 ) {
			msg ="Built luKey: " + luKey + " from strKey: " + strKey + " and elemName: " + elemName;
			try {
				aLog.logAction(msg);
			} catch(Exception el) {
				System.out.println(msg);
			}
		}
		Vector vecTmp = config.getKeyList(luKey);
		if ( util.logLevel > 2 ) {
			msg = "vecTmp: " + vecTmp.toString();
			try {
				aLog.logAction(msg);
			} catch(Exception el) {
				System.out.println(msg);
			}
		}
		Vector vecTmpTypes = config.getValueList(luKey);
		if ( util.logLevel > 2 ) {
			msg = "vecTmpTypes: " + vecTmpTypes.toString();
			try {
				aLog.logAction(msg);
			} catch(Exception el) {
				System.out.println(msg);
			}
		}
		Vector vecTmpTypesAttributes = config.getAttributeList(luKey);
		if ( util.logLevel > 2 ) {
			msg = "vecAttribTypes: " + vecTmpTypesAttributes.toString();
			try {
				aLog.logAction(msg);
			} catch(Exception el) {
				System.out.println(msg);
			}
		}

		
		luKey += "-TRANSLATIONS";
//		System.out.println("LookupKey = "  + luKey);
		Vector vecTransFrom = config.getKeyList(luKey);
//		System.out.println("vecTransFrom = "  + vecTransFrom.toString());
		Vector vecTransTo = config.getValueList(luKey);
//		System.out.println("vecTransTo = "  + vecTransTo.toString());

		String tmpElemName = elemName;
		
		int ret = vecTransFrom.indexOf(tmpElemName);
		if (util.logLevel > 2) {
		System.out.println("tmpElemName found in  "  + vecTransFrom.toString() + " = " + ret);
		}
		if (ret > -1 ) {
			tmpElemName = (String)vecTransTo.elementAt(ret);
		}
		if (util.logLevel > 2) {
		System.out.println("tmpElemName is now = " + tmpElemName);
		}
		
//dc1016		Element childElem = new Element("row");		
//		Element gContent = new Element (tmpElemName);
		Element childElem;
	//	Element childElemS2;
		Element childElemSERVICE;
		if (tmpElemName.contains("diagnosis")) {
			childElem = new Element ("diagnosis");	
		} else if (tmpElemName.contains("treat")) {
			childElem = new Element ("treatmentplan");	
		} else if (tmpElemName.contains("service")) {
			
			//getServiceItem = tmpElemName.substring(8);
			
			System.out.println("service found - " + getServiceItem);
			
			childElem = new Element ("treatmentplan");	
			
			//		childElemS2 = new Element ("treatmentplan");
					

			childElemSERVICE = new Element ("planpart");
			childElemSERVICE.setText( "Service" );			
			childElem.addContent(childElemSERVICE);
			//		childElemS2.addContent(childElemSERVICE);
			
			
			serviceDoc = 1;
			serviceItem = 1;
			serviceItemTotal = 2;
			
		} else {
			childElem = new Element (tmpElemName);
		}
			//childElem.addContent(gContent);
		//Element gchildElem = new Element(tmpElemName);
		
		try{
			if ( util.logLevel > 2 ) {
				msg = "vecTmp size = : " + vecTmp.size();
				try {
					aLog.logAction(msg);
				} catch(Exception el) {
					System.out.println(msg);
				}
			}
			for (int i=0; i<vecTmp.size();i++){
				Element tmpElem = null;
				
				String fldName = "" + (String)vecTmp.elementAt(i);

				String tmpVal = util.getValue(docBack, fldName);
				String tmpValS2 = "";
				
				if(fldName.equals("S17_1") | fldName.equals("S17_2")) {
					System.out.println("service -match s17");					
				} else { 
					if(serviceDoc == 1) {
						tmpVal = docBack.getItemValue(fldName).get(0).toString();
				//		tmpValS2 = docBack.getItemValue(fldName).get(1).toString();
						
						System.out.println("service - tmpval = " + tmpVal);
					}else {
						//tmpVal = util.getValue(docBack, fldName);
						
					}
				}
				
				String tmpElemAttribute = "" + (String)vecTmpTypesAttributes.elementAt(i);
				if (util.logLevel > 2) {
				System.out.println("FLDNAME = " + fldName);
				}
				tmpElemName = "" + (String)vecTmp.elementAt(i);
				if (util.logLevel > 2) {
				System.out.println ("tmpElemName = " + tmpElemName);
				}
				ret = vecTransFrom.indexOf(tmpElemName);
				if (util.logLevel > 2) {
				System.out.println("tmpElemName found in  "  + vecTransFrom.toString() + " = " + ret);
				}
				String orgElemName = tmpElemName;
				if (ret > -1 ) {
					tmpElemName = (String)vecTransTo.elementAt(ret);
				}
				if (util.logLevel > 2) {
				System.out.println("tmpElemName is now = " + tmpElemName);
				}
				String vecTmpTypesString = vecTmpTypes.elementAt(i).toString();
				String vecTmpTypesTruncatedLengthString = vecTmpTypesString.substring(5) ;

				//10-25
				//System.out.println ("vecTmpTypesString: " +vecTmpTypesString );
				
				
				if( vecTmpTypesString.contains("value") ) {
					System.out.println("insideValue nowx tmpVal = " + tmpVal);
//				if( vecTmpTypes.elementAt(i).equals("value") ) {
					// --- This is a field (key+value pair)---
					tmpElem = new Element(tmpElemName);
					
					if (tmpElemName.equals("icd10desccode")) {
						validDiagnosisCode = "";
						System.out.println ("icd10desccodeTOP MHDC started");
						if (tmpVal.equals("") ) {
							System.out.println ("icd10desccodeTOP is not");
							validDiagnosisCode = "notvalid";

						}
					}
					if (tmpElemName.equals("plandetail")) {
						validDiagnosisCode = "";
						System.out.println ("plandetail MHDC started");
						if (tmpVal.equals("") ) {
							System.out.println ("plandetail is not");
							validDiagnosisCode = "notvalid";

						}
					}
					if ( util.logLevel > 2 ) {
						msg ="Create x field element for " + tmpElemName;
						/*10-25
						if (util.logLevel > 2) {
						System.out.println ("Create (value) field element for " + tmpElemName);
						}
						*/
						
						try {
							aLog.logAction(msg);
						} catch(Exception el) {
							System.out.println(msg);
						}
					}
					if (docBack != null ) {
						/*10-25
						if (util.logLevel > 2) {
							System.out.println ("docBack not null " + fldName);
						}
					
					*/
						/*
						if (tmpElemName.equals("rownum")){
							System.out.println ("rownum set value" );

							tmpVal = docCountDOS;
							tmpElem.setText( tmpVal );
						} 
						*/
						
					
						if (docBack.hasItem(fldName)) {
							Item itm = docBack.getFirstItem(fldName);
							if (util.logLevel > 2) {
								
								//10-25 System.out.println (" itm " + fldName + " = " + itm);
								if (fldName.equals("S17_1A")) {
							//		Vector xtemp = docBack.getItemValue("S17_1A");
									//String xString = itm.getValueString(0);
									//String xST = xtemp[0].toString();
									//String[] xarray = docBack.getItemValue("S17);
//									String[] array = xtemp.toArray(new String[xtemp.size()]); 
									
									

									
									System.out.println ("item count1: " + docBack.getItemValue("S17_1A"));
									System.out.println ("item size1: " + docBack.getItemValue("S17_1A").size());
									System.out.println ("item get1: " + docBack.getItemValue("S17_1A").get(0));
									
									
								}
								
							}
							if (itm != null) {
								
								if (util.logLevel > 2) {
									System.out.println ("itm not null " + fldName);
								}
								
								
//verify??10-25								String tmpVal = util.getValue(docBack, fldName);
								if (tmpVal.equals("n/a")) {
									tmpVal = "0";	
									//tmpVal ="N/A";
								} else if (tmpVal.equals("M")) {
									tmpVal = "Male";
								}else if (tmpElemName.equals("icd10desccode")) {
									validDiagnosisCode = "";
									System.out.println ("icd10desccode started");
									if (tmpVal.equals("") ) {
										System.out.println ("icd10desccode is not");
										validDiagnosisCode = "notvalid";

									} else {
										char[] charArray = tmpVal.toCharArray();
										for (int cap=0; i <charArray.length; i++){
											  //if the character is a letter
								            if( Character.isLetter(charArray[i]) ){
								                
								                //if any character is not in upper case, return false
								                if( !Character.isUpperCase( charArray[i] )){
								                    validDiagnosisCode = "notvalid";
								                }
								            }
										}
									}
									
								}else if (tmpVal.equals("F")) {
									tmpVal = "Female";
									
								}else if (tmpVal.equals("St Clair")) {
									tmpVal = "St. Clair";
									                     
								}else if (tmpElemName.equals("mentalstatcognition")) {
									if (tmpVal.contains("Loose")) {
							
										 tmpVal = "Loose Assoc/Disorganized";
									}								}
								
								else if (tmpVal.equals("Re-assessment")) {
									tmpVal = "Reassessment";
									
								} else if (tmpVal.equals("Madison County")) {
									tmpVal = "Madison";
								} else if (tmpElemName.equals("educationlevel") | tmpElemName.equals("EducationLevel")) {
									System.out.println ("inside educationlevel" );
								
										if (tmpVal.substring(0,8).equals("Bachelor")) {
											System.out.println ("inside bach" );
											
											tmpVal = "Bachelor's degree";
										} else if (tmpVal.substring(0,4).equals("Asso")) {
											
											tmpVal = "Associate's degree";
										}  else if (tmpVal.substring(0,4).equals("H.S.")) {
											
											tmpVal = "H.S. diploma/GED";
										}  else if (tmpVal.substring(0,3).equals("Pre")) {
											
											tmpVal = "Pre-K/Kindergarten";
										}  else if (tmpVal.substring(0,3).equals("Tra")) {
											
											tmpVal = "Trade/technical training";
										}  else if (tmpVal.substring(0,3).equals("Mas")) {
											
											tmpVal = "Master's/Doctoral degree";
										} else if (tmpVal.equals("Grade 4 - 5")) {
											tmpVal = "Grade 4-5";
										}else if (tmpVal.equals("Grade 6 - 8")) {
											tmpVal = "Grade 6-8";
										}else if (tmpVal.equals("Grade 9 - 12")) {
											tmpVal = "Grade 9-12";
										}else if (tmpVal.equals("Never Attended")) {
											tmpVal = "Never attended";
										} 
										
								} else if (tmpElemName.equals("parentstate") | tmpElemName.equals("ParentState") | tmpElemName.equals("ecstate") | tmpElemName.equals("EmergencyState")  | tmpElemName.equals("state") |  tmpElemName.equals("ClientState")){
									if(tmpVal.equals("N/A")) {
										tmpVal = "";
									} else if (tmpVal.equals("Illinois")) {
										tmpVal = "IL";
									}else if (tmpVal.equals("Indiana")) {
										tmpVal = "IN";
									}else if (tmpVal.equals("Wisconsin")) {
										tmpVal = "WI";
									}else if (tmpVal.equals("Missouri")) {
										tmpVal = "MO";
									}
								}
								else if (tmpElemName.equals("rownum")){
									tmpVal = String.valueOf(docCountROW);
								} else if (tmpElemName.equals("race")){
									if (tmpVal.equals("10")) {
										tmpVal = "";
									}if (tmpVal.equals("20")) {
										tmpVal = "";
									}
								} else if (tmpElemAttribute.equals("int")){
									
									if (tmpVal.indexOf(".")==-1) {
										
									} else {
										tmpVal = (tmpVal.substring(0, tmpVal.indexOf(".")));
									}
								}
								int vecTmpTypesStringlength = vecTmpTypesString.length();
								
							/*
							  		if (vecTmpTypesStringlength > 5)  {
							 
										
										int vecTmpTypesTruncatedLength = Integer.parseInt(vecTmpTypesTruncatedLengthString);
										
										if(tmpVal.length()> vecTmpTypesTruncatedLength) {
										String vecTmpTypesTruncated = util.getValue(docBack, fldName).substring(0, vecTmpTypesTruncatedLength);
										System.out.println ("Truncated: " + vecTmpTypesTruncated  + " using " + vecTmpTypesTruncatedLength );
										tmpVal = util.getValue(docBack, fldName).substring(0, vecTmpTypesTruncatedLength);
										}
									}
									*/
								
								if (util.logLevel > 2) {
								System.out.println (" tmpVal = " + tmpVal);
								}
								msg = tmpElemName + "(" + fldName + ") = " + tmpVal;
								if (util.logLevel > 2) {
								System.out.println(msg);
								}
								if ( util.logLevel > 4 ) {
									msg = tmpElemName + "(" + fldName + ") = " + tmpVal;
									System.out.println(msg);
									try {
										aLog.logAction(msg);
									} catch(Exception el) {
										System.out.println(msg);
									}
								}
								tmpElem.setText( tmpVal );
								if (tmpElemAttribute.equals("str")){	
									tmpElem.setAttribute("type", "str");
								} else if(tmpElemAttribute.equals("bool")) {
									tmpElem.setAttribute("type", "bool");
								}	else if(tmpElemAttribute.equals("date")) {
									tmpElem.setAttribute("type", "date");
								}	else if(tmpElemAttribute.equals("list")) {
									tmpElem.setAttribute("type", "list");
								}	else if(tmpElemAttribute.equals("int")) {
									tmpElem.setAttribute("type", "int");
								}
							} else {
								
								tmpElem.setText("");
						
								if (tmpElemAttribute.equals("str")){	
									tmpElem.setAttribute("type", "str");
								} else if(tmpElemAttribute.equals("bool")) {
									tmpElem.setAttribute("type", "bool");
								}	else if(tmpElemAttribute.equals("date")) {
									tmpElem.setAttribute("type", "date");
								}	else if(tmpElemAttribute.equals("list")) {
									tmpElem.setAttribute("type", "list");
								}	else if(tmpElemAttribute.equals("int")) {
									tmpElem.setAttribute("type", "int");
								}
							}
						} else {
							if (tmpElemName.equals("rownum")){
								System.out.println ("rownum set value" );

								tmpVal = String.valueOf(docCountROW);
								tmpElem.setText( tmpVal );
							} else {
								tmpElem.setText("");
							};
							if (tmpElemAttribute.equals("str")){	
								tmpElem.setAttribute("type", "str");
							} else if(tmpElemAttribute.equals("bool")) {
								tmpElem.setAttribute("type", "bool");
							}	else if(tmpElemAttribute.equals("date")) {
								tmpElem.setAttribute("type", "date");
							}	else if(tmpElemAttribute.equals("list")) {
								tmpElem.setAttribute("type", "list");
							}	else if(tmpElemAttribute.equals("int")) {
								tmpElem.setAttribute("type", "int");
							}
						}
					} else {
						tmpElem.setText("");
						System.out.println("setting attribute: " + vecTmpTypesAttributes.elementAt(i).toString() );
						if (tmpElemAttribute.equals("str")){	
							tmpElem.setAttribute("type", "str");
						} else if(tmpElemAttribute.equals("bool")) {
							tmpElem.setAttribute("type", "bool");
						}	else if(tmpElemAttribute.equals("date")) {
							tmpElem.setAttribute("type", "date");
						}	else if(tmpElemAttribute.equals("list")) {
							tmpElem.setAttribute("type", "list");
						}	else if(tmpElemAttribute.equals("int")) {
							tmpElem.setAttribute("type", "int");
						}
					}
					
				} else if(vecTmpTypesString.contains("mult")) {
					tmpElem = new Element(tmpElemName);
					tmpVal = "No";
					// vecTmpTypesString = vecTmpTypes.elementAt(i).toString();
					//String vecTmpTypesTruncatedLengthString = vecTmpTypesString.substring(5) ;
					String vecTmpTypesMultField = vecTmpTypesTruncatedLengthString.split("[,]",0)[0];
					String vecTmpTypesMultValue = vecTmpTypesTruncatedLengthString.split("[,]",0)[1].substring(0,vecTmpTypesTruncatedLengthString.split("[,]",0)[1].length()-1);
					
					fldName = vecTmpTypesMultField;
					
					if ( util.logLevel > 2 ) {
						msg ="Create field element for Multivalue " + tmpElemName;
						if (util.logLevel > 2) {
						System.out.println (msg + tmpElemName);
						System.out.println ("Right multi: " + vecTmpTypesTruncatedLengthString);
						System.out.println ("Right multi field: " + vecTmpTypesMultField);
						System.out.println ("Right multi value: " + vecTmpTypesMultValue);
						
						}
						try {
							aLog.logAction(msg);
						} catch(Exception el) {
							System.out.println(msg);
						}
					}
					if (docBack != null ) {
	
						if (docBack.hasItem(fldName)) {
							Item itm = docBack.getFirstItem(fldName);
							if (util.logLevel > 2) {
								System.out.println (" Multi item " + fldName + " = " + itm);
							}
							if (itm != null) {
								String tmpValmulti = util.getValue(docBack, fldName);
								if (util.logLevel > 2) {
									System.out.println ("tmpValMulti = " + tmpValmulti);
								}
								
								if (tmpValmulti.contains(vecTmpTypesMultValue)) { 
										tmpVal = "Yes";
									}
								if (util.logLevel > 2) {
								System.out.println (" tmpVal = " + tmpVal);
								}
								msg = tmpElemName + "(" + fldName + ") = " + tmpVal;
								if (util.logLevel > 2) {
								System.out.println(msg);
								}
								if ( util.logLevel > 4 ) {
									msg = tmpElemName + "(" + fldName + ") = " + tmpVal;
									System.out.println(msg);
									try {
										aLog.logAction(msg);
									} catch(Exception el) {
										System.out.println(msg);
									}
								}
								tmpElem.setText( tmpVal );
								if (tmpElemAttribute.equals("str")){	
									tmpElem.setAttribute("type", "str");
								} else if(tmpElemAttribute.equals("bool")) {
									tmpElem.setAttribute("type", "bool");
								}	else if(tmpElemAttribute.equals("date")) {
									tmpElem.setAttribute("type", "date");
								}	else if(tmpElemAttribute.equals("list")) {
									tmpElem.setAttribute("type", "list");
								}
							} else {
								tmpElem.setText("");
								if (tmpElemAttribute.equals("str")){	
									tmpElem.setAttribute("type", "str");
								} else if(tmpElemAttribute.equals("bool")) {
									tmpElem.setAttribute("type", "bool");
								}	else if(tmpElemAttribute.equals("date")) {
									tmpElem.setAttribute("type", "date");
								}	else if(tmpElemAttribute.equals("list")) {
									tmpElem.setAttribute("type", "list");
								}
							}
						} else {
							tmpElem.setText("");
							if (tmpElemAttribute.equals("str")){	
								tmpElem.setAttribute("type", "str");
							} else if(tmpElemAttribute.equals("bool")) {
								tmpElem.setAttribute("type", "bool");
							}	else if(tmpElemAttribute.equals("date")) {
								tmpElem.setAttribute("type", "date");
							}	else if(tmpElemAttribute.equals("list")) {
								tmpElem.setAttribute("type", "list");
							}
						}
					} else {
						tmpElem.setText("");
						System.out.println("setting attribute: " + vecTmpTypesAttributes.elementAt(i).toString() );
						if (tmpElemAttribute.equals("str")){	
							tmpElem.setAttribute("type", "str");
						} else if(tmpElemAttribute.equals("bool")) {
							tmpElem.setAttribute("type", "bool");
						}	else if(tmpElemAttribute.equals("date")) {
							tmpElem.setAttribute("type", "date");
						}	else if(tmpElemAttribute.equals("list")) {
							tmpElem.setAttribute("type", "list");
						}
					}
					
				
				} else if(vecTmpTypes.elementAt(i).equals("configdoc")) {
					tmpElem = new Element(tmpElemName);
					Vector vecConfigdocValue = config.getParameterList("XML-IHFS-CONFIGDOC-" + fldName);
					//Vector vecTmp = config.getKeyList(luKey);
					//if ( vecTmp == null )
					//	return null;	
					//String elemName = (String)vecTmp.elementAt(0);
					
//					(String)vecTmp.elementAt(0);
					String tmpConfigdocValue = (String)vecConfigdocValue.elementAt(0);
					//lotus.domino.Document docConfigValue;
					//docConfigValue =  config.getConfigDoc("XML-IHFS-CONFIGDOC-HFSXMLVER");

					if (util.logLevel > 2) {
						System.out.println ("Configdoc lookup " + strKey + " fieldname " + tmpElemName);
						System.out.println ("Configdoc value " + tmpConfigdocValue);

					}
					try {
						aLog.logAction(msg);
					} catch(Exception el) {
						System.out.println(msg);
					}

					tmpElem.setText(tmpConfigdocValue);
					
					if (tmpElemAttribute.equals("str")){	
						tmpElem.setAttribute("type", "str");
					} else if(tmpElemAttribute.equals("bool")) {
						tmpElem.setAttribute("type", "bool");
					}	else if(tmpElemAttribute.equals("date")) {
						tmpElem.setAttribute("type", "date");
					}	else if(tmpElemAttribute.equals("list")) {
						tmpElem.setAttribute("type", "list");
					}
					
					
				} else if( vecTmpTypes.elementAt(i).equals("value+masterdate") ) {
					
					
					// --- This is a field (key+value pair)---
					// --- Add the util.masterdate string to the field value ---
					tmpElem = new Element(tmpElemName);
					if ( util.logLevel > 3 ) {
						msg ="Create field element for " + tmpElemName;
						if (util.logLevel > 2) {
							System.out.println ("Create field element for " + tmpElemName);
						}
						try {
							aLog.logAction(msg);
						} catch(Exception el) {
							System.out.println(msg);
						}
					}
					if (docBack != null ) {
						if (docBack.hasItem(fldName)) {
							Item itm = docBack.getFirstItem(fldName);
						
							/*10-25
							 if (util.logLevel > 2) {
								System.out.println (" itm " + fldName + " = " + itm);
							}
							*/
							
							if (itm != null) {
								tmpVal = util.getValue(docBack, fldName) + "_" + util.masterDate;
								if (util.logLevel > 2) {
								System.out.println (" tmpVal = " + tmpVal);
								}
								msg = tmpElemName + "(" + fldName + ") = " + tmpVal;
								if (util.logLevel > 2) {
								System.out.println(msg);
								}
								if ( util.logLevel > 4 ) {
									msg = tmpElemName + "(" + fldName + ") = " + tmpVal;
									System.out.println(msg);
									try {
										aLog.logAction(msg);
									} catch(Exception el) {
										System.out.println(msg);
									}
								}
								tmpElem.setText( tmpVal );
								System.out.println("setting attribute: " + tmpElemAttribute );
								if (tmpElemAttribute.equals("str")){	
									tmpElem.setAttribute("type", "str");
								} else if(tmpElemAttribute.equals("bool")) {
									tmpElem.setAttribute("type", "bool");
								}	else if(tmpElemAttribute.equals("date")) {
									tmpElem.setAttribute("type", "date");
								}	else if(tmpElemAttribute.equals("list")) {
									tmpElem.setAttribute("type", "list");
								}								
								
							} else {
								tmpElem.setText("");
								if (tmpElemAttribute.equals("str")){	
									tmpElem.setAttribute("type", "str");
								} else if(tmpElemAttribute.equals("bool")) {
									tmpElem.setAttribute("type", "bool");
								}	else if(tmpElemAttribute.equals("date")) {
									tmpElem.setAttribute("type", "date");
								}	else if(tmpElemAttribute.equals("list")) {
									tmpElem.setAttribute("type", "list");
								}
							}
						} else {
							tmpElem.setText("");
							if (tmpElemAttribute.equals("str")){	
								tmpElem.setAttribute("type", "str");
							} else if(tmpElemAttribute.equals("bool")) {
								tmpElem.setAttribute("type", "bool");
							}	else if(tmpElemAttribute.equals("date")) {
								tmpElem.setAttribute("type", "date");
							}	else if(tmpElemAttribute.equals("list")) {
								tmpElem.setAttribute("type", "list");
							}
						}
					} else {
						tmpElem.setText("");
						if (tmpElemAttribute.equals("str")){	
							tmpElem.setAttribute("type", "str");
						} else if(tmpElemAttribute.equals("bool")) {
							tmpElem.setAttribute("type", "bool");
						}	else if(tmpElemAttribute.equals("date")) {
							tmpElem.setAttribute("type", "date");
						}	else if(tmpElemAttribute.equals("list")) {
							tmpElem.setAttribute("type", "list");
						}
					}
					
				} else if( vecTmpTypes.elementAt(i).equals("container") ) {
					// ---this is a container object---
					String tmpKey = strKey.toUpperCase();
					if (tmpKey.equals("")) {
						tmpKey += elemName.equals("") ? "" : elemName.toUpperCase();
					} else {
						tmpKey += elemName.equals("") ? "" : "-" + elemName.toUpperCase();
					}
					
					if ( util.logLevel > 2 ) {
						msg ="Create container element " + i + " of " + vecTmp.size() + " as child of " + strKey + " using: " + elemName;
						try {
							aLog.logAction(msg);
						} catch(Exception el) {
							System.out.println(msg);
						}
					}
					tmpElem = (Element)buildElementFromConfig(docBack, tmpKey, orgElemName, 1);
					if ( util.logLevel > 1 ) {
						msg ="Done creating container element, this was element " + i + " of " + vecTmp.size();
						try {
							aLog.logAction(msg);
						} catch(Exception el) {
							System.out.println(msg);
						}
					}
				} else {
					// ---This is an unknown type ---
					// shoot out an error email, we don't know how to handle this 'type'
					if ( util.logLevel > 0 )
						System.out.println ("!!! Unknown value type !!!");
					sendErrorNotice( new Exception("!!! Unknown value type !!!"), dbThis, "Module: buildElementFromConfig processing Element: " + tmpElemName + " flagged as type: " + vecTmpTypes.elementAt(i) );
					isErr = true;
					return null;
				}
				if ( tmpElem == null ) {
					if ( util.logLevel > 0 )
						System.out.println ("!!! tmpElem is null !!!");
						sendErrorNotice( new Exception("!!! tmpElem is null !!!"), dbThis, "Module: buildElementFromConfig processing Element: " + tmpElemName + " flagged as type: " + vecTmpTypes.elementAt(i) );
				} else {
					if ( util.logLevel > 0 ) {
						msg ="tmpElem: " + tmpElem.getName() + " = " + tmpElem.getText() + " => " + tmpElem.toString();
						try {
							aLog.logAction(msg);
						} catch(Exception el) {
							System.out.println(msg);
						}
						msg ="childElem: " + childElem.getName() + " = " + childElem.getText() + " => " + childElem.toString();
						try {
							aLog.logAction(msg);
						} catch(Exception el) {
							System.out.println(msg);
						}
					}
					if (childElem == null) {
						
						childElem = tmpElem;
					} else {
						
						//gContent.addContent(tmpElem);
						//childElem.addContent(gContent);
						//destDocument.getRootElement().getChild("result").addContent(resultEle.get(count).detach());
							
							childElem.addContent(tmpElem);
							
						/*10-18-20	if (fldName.equals("S17_1A")) {
								childElem.addContent(tmpElem);
							}
							*/
							
						//childElem.addContent(gContent).detach();
					}
				}
			}		//	forall element names in vecTmp
		}
		catch (Exception e){
			//error handling here
			sendErrorNotice( new Exception("General error encountered in buildElementFromConfig"), dbThis, "Module: buildElementFromConfig." );
			if ( util.logLevel > 0 ) {
				msg ="General error encountered in buildElementFromConfig: " + e.getMessage();
				try {
					aLog.logError(e.hashCode(), msg);
				} catch(Exception el) {
					System.out.println(msg);
				}
			}
			e.printStackTrace();
			isErr = true;
		}
		if ( util.logLevel > 0 ) {
			msg ="Finished creating childElem: " + childElem.toString();
			try {
				aLog.logAction(msg);
			} catch(Exception el) {
				System.out.println(msg);
			}
		}
		/*
		Element nextR = new Element("rowx");					
		nextR.addContent(childElem);
		return nextR;
		*/
		
		return childElem;
	
	}

	
	public static void applyattribute (String getattribute) {
		if (getattribute == "str") {
//			tmpElem.setAttribute("type", "str");
		} else if (getattribute == "none"){
			
		}
	}

	

public static void sendErrorNotice(Exception e, Database dbThis, String errMsg) {
	try {

		lotus.domino.Document docMemo = dbThis.createDocument();
		docMemo.replaceItemValue("Form", "ErrorInfo");
		//docMemo.replaceItemValue("SendTo","dchapman@teamcentric.com");
		//docMemo.replaceItemValue("SendTo", config.getErrorNotifyTo());
		//docMemo.replaceItemValue("CopyTo", config.getErrorNotifyCopyTo());
		docMemo.replaceItemValue("Subject", "Error encountered in " + dbThis.getTitle() + " (" + dbThis.getFilePath() + ") on server " + dbThis.getServer());
		Item itm = docMemo.replaceItemValue("Body", errMsg );
		if ( e != null ) {
			itm.appendToTextList( e.getMessage() );
		}
		//docMemo.send( false );
		docMemo.save();
		/*		
		lotus.domino.Document docMemo = dbThis.createDocument();
		docMemo.replaceItemValue("Form", "Memo");
		docMemo.replaceItemValue("SendTo","dchapman@teamcentric.com");
		//docMemo.replaceItemValue("SendTo", config.getErrorNotifyTo());
		//docMemo.replaceItemValue("CopyTo", config.getErrorNotifyCopyTo());
		docMemo.replaceItemValue("Subject", "Error encountered in " + dbThis.getTitle() + " (" + dbThis.getFilePath() + ") on server " + dbThis.getServer());
		Item itm = docMemo.replaceItemValue("Body", errMsg );
		if ( e != null ) {
			itm.appendToTextList( e.getMessage() );
		}
		docMemo.send( false );
		*/
	}
	catch (Exception err) {
		isErr = true;
		msg ="General error sending error notice email: " + errMsg;
		try {
			aLog.logError(err.hashCode(), msg);
		} catch(Exception el) {
			System.out.println(msg);
		}
		err.printStackTrace();
	}
}

public static boolean backOutBatch( Utilities util, Database dbThis )
{
	boolean results = false;
	try {
		// - Remove all processed documents from the document collection -
		Vector vecTmp = new Vector();
		vecTmp.add("«»");
		dcProcessed = vwFolder.getAllDocumentsByKey(vecTmp, true);

		//delete the batch file
		results = deleteFile( util.xmlFile );
		
		if ( results )
		{
			if ( util.logLevel > 0 ) {
				msg ="Batch file deleted.";
				try {
					aLog.logAction(msg);
				} catch(Exception el) {
					System.out.println(msg);
				}
			}
		}
		else
		{
			if ( util.logLevel > 0 ) {
				msg ="Batch file NOT deleted.";
				try {
					aLog.logError((int)1698, msg);
				} catch(Exception el) {
					System.out.println(msg);
				}
			}
			sendErrorNotice( new Exception("Batch file NOT deleted."), dbThis, "Module: Back out batch processing." );
		}
		results = deleteFile( util.tokenFile );
		if ( results )
		{
			if ( util.logLevel > 0 ) {
				msg ="Trigger file deleted.";
				try {
					aLog.logAction(msg);
				} catch(Exception el) {
					System.out.println(msg);
				}
			}
		}
		else
		{
			if ( util.logLevel > 0 ) {
				msg ="Trigger file NOT deleted.";
				try {
					aLog.logError((int)1699, msg);
				} catch(Exception el) {
					System.out.println(msg);
				}
			}
			sendErrorNotice( new Exception("Trigger file NOT deleted."), dbThis, "Module: Back out batch processing." );
		}
		
	}	//try
	catch(Exception e) 
	{
		if ( util.logLevel > 0 ) {
			msg ="Module: Back out batch processing..";
			try {
				aLog.logAction(msg);
			} catch(Exception el) {
				System.out.println(msg);
			}
		}
		sendErrorNotice( e, dbThis, "Module: Back out batch processing." );
		e.printStackTrace();
		isErr = true;
	}
	return results;
}

public static boolean deleteFile( String fileToKill )
{
	boolean deleteSuccess = false;
	try {
		if (util.logLevel > 0) {
			msg ="Deleting file: " + fileToKill;
			try {
				aLog.logAction(msg);
			} catch(Exception el) {
				System.out.println(msg);
			}
		}
		deleteSuccess = (new File( fileToKill )).delete();
	} catch (Exception e) {
		isErr = true;
		sendErrorNotice( e, dbThis, "Module: deleteFile: " + fileToKill );
		e.printStackTrace();
	}
	return deleteSuccess;
}


}