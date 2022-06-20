
Impression Zebra en X++
Dernière modification : 18/12/2020
Contexte
Dans le cadre du projet Tréfilunion, nous avons rencontré une problématique d'impression d'étiquettes sur des imprimantes Zebra
 
Plusieurs options ont été étudiées :
Génération d'un état SSRS contenant l'étiquette
Option 1 : Impression via un logiciel tiers (à définir) capable de transformer l'impression au format ZPL
Option 2 : Impression via un Driver Zebra capable de transformer l'impression au format ZPL
Option 3 : Impression en PDF direct sur imprimante Zebra equipée Link OS
Envoi via webservice via une solution externe type Bartender ou CodeSoft/sentinel
Impression via X++ en ZPL
 
C'est la troisième solution d'impression en ZPL via X++ qui a été retenue.
 
La partie la plus technique concernait l'impression de Logos sur l'étiquette, une classe a été développée pour traiter ce point. (Voir plus bas)
 
Ci-dessus une description des points principaux utilisés pour la mise en oeuvre de cette impression
Fonction d'impression Zebra
Il existe une fonction standard permettant d'imprimer une chaine de texte "ZPL" directement sur une imprimante Zebra.
Cette imprimante aura été au préalable paramétrée via le DRA (Document Routing Agent)
 
Commande X++ :
WHSDocumentRouting::printLabelToPrinter(PrinterName, ZPL);
 
Masque d'impression
Pour le stockage des masques d'impression nous nous sommes appuyé sur le standard de la gestion d’entrepôt.
Ecran  « Gestion des entrepôts > Paramétrage > Acheminement de document > Mises en page de l'acheminement de document » :


 
 
Classe d'impression d'images
Description
Une classe SICSysZPLImageConverter permettant de convertir une image au format exploitable en ZPL à été développée.
Cette classe permet de convertir une image BMP, JPG, etc  au format ZPL (balise GFA)
Elle permet également
la compression
la rotation
le redimensionnement de l'image en conservant le rapport H/L.
 
 
Exemple d'utilisation de SICSysZplImageConverter :
 
        SICSysZPLImageConverter imageConverter = new SICSysZPLImageConverter();
  
        imageConverter.setCompressHex(true);
        imageConverter.setBlacknessLimitPercentage(50);
        
        if (sicProdParameters.LabelLogoRotate)
        {​​​​​​
            imageConverter.parmRotateType(System.Drawing.RotateFlipType::Rotate90FlipNone);
        }​​​​​​
        imageConverter.parmNewWidth(sicProdParameters.LabelLogoCompanyWidth);
        imageConverter.parmNewHeight(sicProdParameters.LabelLogoCompanyHeight);

        logoStr = imageConverter.getGraphicFieldContainer(companyImage.Image);
 
 
 
Classe SICSysZPLImageConverter :
 
/// <summary>
///        Image to ZPL converter class
/// </summary>
public class SICSysZPLImageConverter
{​​
    private int blackLimit = 380;
    private int total;
    private int widthBytes;
    private boolean compressHex = false;
    private static Map mapCode = new Map(Types::Integer, Types::String);
    System.Drawing.RotateFlipType rotateType;
    private int newWidth, newHeight ;

    public const real pixelEnCm = 37.7953;


 
    public int parmNewWidth(int _newWidth = newWidth)
    {​​
        newWidth = _newWidth;

        return newWidth;
    }​​

    public int parmNewHeight(int _newHeight = newHeight)
    {​​
        newHeight = _newHeight;

        return newHeight;
    }​​

    public int parmRotateType(int _rotateType = rotateType)
    {​​
        rotateType = _rotateType;

        return rotateType;
    }​​

    /// <summary>
    ///        Convert image and surround with header end footer document commands
    /// </summary>
    /// <param name = "image">blob image</param>
    /// <returns>Converted ZPL from image with header and footer </returns>
    public str convertfromImg(System.Drawing.Bitmap image)
    {​​        
        return this.headDoc() + this.getGraphicField(image) + this.footDoc();
    }​​

    /// <summary>
    ///        Convert image to heaxdecimal
    /// </summary>
    /// <param name = "orginalImage">blob image</param>
    /// <returns>Converted image in hexadecimal format</returns>
    private str createBody(System.Drawing.Bitmap orginalImage)
    {​​
        System.Text.StringBuilder    sb = new System.Text.StringBuilder();
        System.Drawing.Color        pixel;
        container                    auxBinaryChar;
        str                            strPixelValue;
        real                        originalRatio, currentRatio;;

        orginalImage = new System.Drawing.Bitmap(orginalImage);

        // Resizing Image
        if (newWidth && newHeight)
        {​​
            originalRatio = orginalImage.Width / orginalImage.Height;

            currentRatio = newWidth / newHeight; 

            if (currentRatio < originalRatio)
            {​​
                newHeight = real2int(newWidth / originalRatio);
            }​​
            else
            {​​
                newWidth = real2int(newHeight * originalRatio);
            }​​


            orginalImage = new System.Drawing.Bitmap(orginalImage, newWidth, newHeight);
        }​​

        
        if (rotateType != System.Drawing.RotateFlipType::RotateNoneFlipNone)
        {​​
            orginalImage.RotateFlip(rotateType);
        }​​

        int width                            = orginalImage.Width;
        int height                            = orginalImage.Height;
        
        int red, green, blue, pixelIndex    = 0;

        for (int i = 1; i <= 8; i++)
        {​​
            auxBinaryChar = conIns(auxBinaryChar, i, System.Char::Parse('0'));
        }​​

        if(width mod 8 > 0)
        {​​
            widthBytes = ((width div 8) + 1);
        }​​
        else
        {​​
            widthBytes = width div 8;
        }​​

        this.total = widthBytes * height;

        for (int h = 0; h<height; h++)
        {​​
            for (int w = 0; w < width; w++)
            {​​
                orginalImage.GetPixel(w, h);

                pixel = orginalImage.GetPixel(w, h);

                red = pixel.R & 0x000000FF;
                green = pixel.G & 0x000000FF;
                blue = pixel.B & 0x000000FF;
                System.Char auxChar = System.Char::Parse('1');

                int totalColor = red + green + blue;
                if (totalColor > blackLimit)
                {​​
                    auxChar = System.Char::Parse('0');
                }​​
                pixelIndex++;
                auxBinaryChar = conPoke(auxBinaryChar, pixelIndex, auxChar);

                if (pixelIndex == 8 || w == (width - 1))
                {​​
                    System.String  auxBinaryStr;
                    for (int i = 1; i <= 8; i++)
                    {​​
                        auxBinaryStr = auxBinaryStr + conPeek(auxBinaryChar, i).ToString();
                    }​​

                    strPixelValue =  this.fourByteBinary(auxBinaryStr);
                    sb.Append(strPixelValue);
                    
                    for (int i = 1; i <= 8; i++)
                    {​​
                        auxBinaryChar = conPoke(auxBinaryChar, i, System.Char::Parse('0'));
                    }​​
                            
                    pixelIndex = 0;
                }​​
            }​​
            sb.Append('\n');
        }​​
        return sb.ToString();
    }​​

    private str fourByteBinary(System.String binaryStr)
    {​​
        System.Int32 decimalValue = System.Convert::ToInt32(binaryStr, 2);
        str retValue;

        if (decimalValue > 15)
        {​​
            retValue =  decimalValue.ToString('X').ToUpper();
        }​​
        else
        {​​
            retValue =  '0' + decimalValue.ToString('X').ToUpper();
        }​​

        return retValue;
    }​​

    private System.String encodeHexAscii(System.String code)
    {​​
        int maxlinea = widthBytes * 2;
        System.Text.StringBuilder sbCode = new System.Text.StringBuilder();
        System.Text.StringBuilder sbLinea = new System.Text.StringBuilder();
        System.String previousLine = null;
        int counter = 1;
        char aux = subStr(code, 1, 1); //code[0];
        boolean firstChar = false;
        for (int i = 2; i <= code.Length; i++)
        {​​
            if (firstChar)
            {​​
                aux = subStr(code, i, 1); //: code[i];
                firstChar = false;
                continue;
            }​​
            if (subStr(code, i, 1) == '\n')
            {​​
                if (counter >= maxlinea && aux == '0')
                {​​
                    sbLinea.Append(',');
                }​​
                else if (counter >= maxlinea && aux == 'F')
                {​​
                    sbLinea.Append('!');
                }​​
                else if (counter > 20)
                {​​
                    int multi20 = (counter div 20) * 20; // TODO a valider
                    int resto20 = (counter mod 20);
                    sbLinea.Append(mapCode.lookup(multi20));
                    if (resto20 != 0)
                    {​​
                        sbLinea.Append(mapCode.lookup(resto20) + aux);
                    }​​
                    else
                    {​​
                        sbLinea.Append(aux);
                    }​​
                }​​
                else
                {​​
                    sbLinea.Append(mapCode.lookup(counter) + aux);
                }​​
                counter = 1;
                firstChar = true;
                if (sbLinea.ToString().Equals(previousLine))
                {​​
                    sbCode.Append(':');
                }​​
                else
                {​​
                    sbCode.Append(sbLinea.ToString());
                }​​
                previousLine = sbLinea.ToString();
                sbLinea.Length = 0;
                continue;
            }​​
            if (aux == subStr(code, i, 1))
            {​​
                counter++;
            }​​
            else
            {​​
                if (counter > 20)
                {​​
                    int multi20 = (counter div 20) * 20;
                    int resto20 = (counter mod 20);
                    sbLinea.Append(mapCode.lookup(multi20));
                    if (resto20 != 0)
                    {​​
                        sbLinea.Append(mapCode.lookup(resto20) + aux);
                    }​​
                    else
                    {​​
                        sbLinea.Append(aux);
                    }​​
                }​​
                else
                {​​
                    sbLinea.Append(mapCode.lookup(counter) + aux);
                }​​
                counter = 1;
                aux = subStr(code, i, 1);
            }​​
        }​​
        return sbCode.ToString();
    }​​

    private str headDoc()
    {​​
        str ret = '^XA \r\n';

        ret = ret + '^FO0,0';

        return ret;
    }​​

    public str getGraphicFieldContainer(container _image)
    {​​
        str        ret;

        Binary imageBinary            = Binary::constructFromContainer(_image);
        
        System.IO.MemoryStream stream = imageBinary.getMemoryStream();
  
        System.Drawing.Bitmap orginalImage = new System.Drawing.Bitmap(stream);

        ret = this.getGraphicField(orginalImage);

        return ret;
  
    }​​

    public str getGraphicField(System.Drawing.Bitmap image)
    {​​
        str        imgZPL;
        str        ret;

        imgZPL = this.createBody(image);

        if(compressHex)
        {​​
            imgZPL = this.encodeHexAscii(imgZPL);
        }​​

        ret = '^GFA,' + int2Str(total) + ',' + int2Str(total) + ',' + int2Str(widthBytes) + ', ' + imgZPL;

        return ret;
  
    }​​

    private str footDoc()
    {​​
        str ret = '^FS\r\n';

        ret = ret +'^XZ';

        return ret;
    }​​

    public void setCompressHex(boolean _compressHex)
    {​​
        this.compressHex = _compressHex;
    }​​

    public void setBlacknessLimitPercentage(int percentage)
    {​​
        blackLimit = real2int(percentage * 768 / 100);
    }​​

    /// <summary>
    ///        Main method for testing purposes
    /// </summary>
    /// <param name = "_args">Args _args</param>
    public static void main(Args _args)
    {​​
        str bitmapFilePath = @'C:\temp\TrefilunoinBMPMono.bmp';
        Companyinfo  companyInfo    = CompanyInfo::find(CompanyInfo::current());
        CompanyImage companyImage    = CompanyImage::find(companyInfo.DataAreaId, companyInfo.TableId, companyInfo.RecId, false, CompanyImageType::CompanyLogo);
        
        container imageContainer    = companyImage.Image;

        Binary imageBinary            = Binary::constructFromContainer(imageContainer);
        
        System.IO.MemoryStream stream = imageBinary.getMemoryStream();
        
        System.Drawing.Bitmap orginalImage = new System.Drawing.Bitmap(stream);
        
        SICSysZPLImageConverter zp = new SICSysZPLImageConverter();
        zp.setCompressHex(true);
        zp.setBlacknessLimitPercentage(50);

        str zplImageStr = zp.convertfromImg(orginalImage);

        System.IO.StreamWriter sr = new System.IO.StreamWriter(@'C:\temp\logoImageXPP3.zpl', false, System.Text.Encoding::Default);

        sr.Write(zplImageStr);
        sr.Close();
    }​​

    /// <summary>
    ///        Constructor
    /// </summary>
    public void new()
    {​​
        this.initMapCode();
    }​​

    /// <summary>
    ///        Init MapCode
    ///        MapCode is used to compress Ascii format
    /// </summary>
    private void initMapCode()
    {​​
        mapCode.insert(1, 'G');
        mapCode.insert(2, 'H');
        mapCode.insert(3, 'I');
        mapCode.insert(4, 'J');
        mapCode.insert(5, 'K');
        mapCode.insert(6, 'L');
        mapCode.insert(7, 'M');
        mapCode.insert(8, 'N');
        mapCode.insert(9, 'O');
        mapCode.insert(10, 'P');
        mapCode.insert(11, 'Q');
        mapCode.insert(12, 'R');
        mapCode.insert(13, 'S');
        mapCode.insert(14, 'T');
        mapCode.insert(15, 'U');
        mapCode.insert(16, 'V');
        mapCode.insert(17, 'W');
        mapCode.insert(18, 'X');
        mapCode.insert(19, 'Y');
        mapCode.insert(20, 'g');
        mapCode.insert(40, 'h');
        mapCode.insert(60, 'i');
        mapCode.insert(80, 'j');
        mapCode.insert(100, 'k');
        mapCode.insert(120, 'l');
        mapCode.insert(140, 'm');
        mapCode.insert(160, 'n');
        mapCode.insert(180, 'o');
        mapCode.insert(200, 'p');
        mapCode.insert(220, 'q');
        mapCode.insert(240, 'r');
        mapCode.insert(260, 's');
        mapCode.insert(280, 't');
        mapCode.insert(300, 'u');
        mapCode.insert(320, 'v');
        mapCode.insert(340, 'w');
        mapCode.insert(360, 'x');
        mapCode.insert(380, 'y');
        mapCode.insert(400, 'z');
    }​​

}​​
 
 
Origine du code de la classe
 
Cette classe est le résultat d'une conversion Java vers X++ du code disponible ici :
http://www.jcgonzalez.com/java-image-to-zpl-example
 
Elle a été améliorée pour gérer la rotation et le redimensionnement des images.
Test du ZPL
Simulation d'impression
Pour tester sans imprimante Zebra un site web qui permet de simuler l'impression et d'avoir un rendu visuel :
 
http://labelary.com/viewer.html
 
Upload pour test
 
Nous avons aussi développé un paramètre d'URL debug pour permettre d'uploader le fichier d'impresssion ZPL et simplifier les tests. A rajouter dans l'URL "&debug=true"
 
Paramètre d'URL :
// for debugging purpose send file when debug URL parameter is set to any value
debug = System.Web.HttpUtility::ParseQueryString(URLUtility::getUrl()).Get('debug');
if (debug)
{​​​
    this.uploadZPLFileToUser(ZPL);
}​​​​​​​​​​
 
Fonction d'upload :
 
private void uploadZPLFileToUser(str _ZPL)
{​​​​​​​​​​
    utcdatetime curDateTime = DateTimeUtil::utcNow();
    var stream = new System.IO.MemoryStream();
    var writer = new System.IO.StreamWriter(stream);
    writer.Write(_ZPL);
    writer.Flush();
    stream.Position = 0;

    curDateTime = DateTimeUtil::applyTimeZoneOffset(curDateTime, DateTimeUtil::getClientMachineTimeZone());

    File::SendFileToUser(stream, strFmt('ZPLPFSF_%1.txt',DateTimeUtil::toStr(curDateTime)));
}​​​​​​​​​​
Rendu final
Pour information voici le rendu final d'une des étiquettes imprimées via X++ :
 
Langage ZPL

 
Manuel ZPL
 
https://www.zebra.com/content/dam/zebra/manuals/printers/common/programming/zpl-zbi2-pm-en.pdf
 
Tips:
Caractères spéciaux :
Pour gérer la caractères spéciaux et accentués ("°é`è@" ... ), utiliser la commande CI28 pour que l'interprétation soit gérée en UTF8 au début du fichier ZPL : 
^FX Use UTF-8
^CI28
 

